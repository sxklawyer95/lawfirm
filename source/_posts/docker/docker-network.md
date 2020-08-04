---
title: Docker 网络
tags: 
  - Docker
date: 2019-03-27
---

> 本文为学习笔记，内容来自于 [《每天五分钟玩转 Docker 容器技术》](https://item.jd.com/12200103.html)

## 目标

1. Docker 提供的几种原生网络
2. 创建自定义网络
3. 容器之间如何通信
4. 容器与外界如何交互

## 原生网络

> Docker 在安装时会自动在 host 上创建三个网络，使用命令 `docker network ls` 查看。得到如下输出：

```
NETWORK ID          NAME                DRIVER              SCOPE

c760fab58f72        bridge              bridge              local
d42648b05ee4        host                host                local
fcc455e2c312        none                null                local
```

由上述内容可知，docker 提供了三个原生网络，分别是： `bridge`、`host`、`none`

### none 网络

顾名思义，none 网络就是什么都没有的网络，启动容器时，如果使用 `--network=none` 来指定使用 `none` 网络，那么该容器的网络环境便是封闭的，而封闭，意味着隔离。挂在这个网络下的除了 `lo`，没有其他任何网卡。

一般来说，对于一些安全性要求较高，并且不需要联网的应用可以使用 `none` 网络。

**注**： `lo` 及 `localhost` 缩写。

### host 网络

通过 `--network=host` 指定容器使用 `host` 网络。**连接到 host 网络的容器共享 Docker host(即安装 Docker 应用的主机) 的网络栈。**

在宿主机上执行命令 `ip l` 得到如下输出：

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 98:ee:cb:6e:55:b3 brd ff:ff:ff:ff:ff:ff
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:d2:c5:c5:23 brd ff:ff:ff:ff:ff:ff
5: vpn0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1418 qdisc fq_codel state UP mode DEFAULT group default qlen 500
    link/none 
30: veth8bd9ae5@if29: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default 
    link/ether 4e:6c:2c:2d:68:4f brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

执行以下命令运行 `busybox` 容器：

```sh
$ docker run -it --network=host busybox
```

进入容器后执行命令 `ip l` 得到如下输出：

```
/ # ip l
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel qlen 1000
    link/ether 98:ee:cb:6e:55:b3 brd ff:ff:ff:ff:ff:ff
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue 
    link/ether 02:42:d2:c5:c5:23 brd ff:ff:ff:ff:ff:ff
5: vpn0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1418 qdisc fq_codel qlen 500
    link/[65534] 
30: veth8bd9ae5@if29: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue master docker0 
    link/ether 4e:6c:2c:2d:68:4f brd ff:ff:ff:ff:ff:ff
```

通过两者输出的内容我们可以确定指定使用 `host` 网路的容器确实使用了和宿主机相同的网络栈。

使用 `host` 机网络有如下几个点需要清楚：

1. 使用该网络最大的优点就是性能，如果容器对网络传输效率有比较高的要求，可以选择使用 host 网络
2. 但如果使用该网络模式，那么必然会损失一些灵活性，比如端口冲突问题，在宿主机上使用过的端口，就无法再在容器中使用了

### bridge 网络

Docker 安装时会默认创建一个名称为 docker0 的 Linux bridge。如果不指定 `--network` 参数，创建的容器默认都会挂到 docker0 上。

使用命令 `brctl show` 查看宿主机上 bridge 网络的情况，得到如下输出：

```
bridge name bridge id       STP enabled interfaces
docker0     8000.0242d2c5c523   no  
```

可以观察到，目前 docker0 上没有挂载任何网络设备。我们执行如下命令运行 busybox:

```sh
$ docker run -it busybox
```

继续输入命令 `brctl show` 查看 bridge 网络情况：

```
bridge name bridge id       STP enabled interfaces
docker0     8000.0242d2c5c523   no      veth4317eb1
```

我们惊喜的发现 interfaces 下显示了一个网卡地址，这个地址就是容器的虚拟网卡，名为 `veth4317eb1`。

在运行的 busybox 容器中执行命令 `ip a` 得到如下输出：

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
31: eth0@if32: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

我们会发现，除了原本有的　`lo` 网卡，还多出了一个名为 `eth0@if32` 的网卡。

在这段内容里面，关于 `eth0@if32` 和 `veth4317eb1` 的关系，实际上，是一对 `veth pair`。

**`veth pair` 是一种成对出现的特殊网络设备，想象成一对由一根虚拟网线连接起来的网卡，网卡的一头是 `eth0@if32`，它身处容器之中，而另一头 `veth4317eb1` 则挂在网桥 docker0 上。**

执行命令 `docker network inspect bridge` 查看到如下内容：

![](https://sherlockblaze.com/resources/img/profession/docker/docker-network-inspect-bridge.png)

我们注意到在 `Config` 配置项下，有两个配置项，分别是 `subnet` 和 `Gateway`，看到网关的值为 `172.17.0.1`，而这个网关，就是我们的 `docker0`。

此时容器的网络拓扑结构如下图所示：

![](https://sherlockblaze.com/resources/img/profession/docker/docker-bridge-config.png)

## user-defined 网络

docker 这么灵活好用的玩意，当然不止可以使用原生网络，用户可以自己根据业务需要创建网络。称为 `user-defined` 网络。

我们可以使用以下命令创建网络：

```sh
$ docker network create --driver bridge my_net
```

使用上述命令我们创建一个名称为 my_net 的 bridge 网络。分别执行命令 `brctl show` 和 `docker network inspect my_net` 可以得到以下输出结果。

```sh
$ brctl show

bridge name bridge id       STP enabled interfaces
br-57b3ea2cc45d     8000.02423e9e1e01   no      
docker0     8000.0242d2c5c523   no      veth4317eb1
```

```sh
$ docker network inspect my_net
```

![](https://sherlockblaze.com/resources/img/profession/docker/docker-network-inspect-my_net.png)

上图中，`172.21.0.0/16` 是 Docker 自动分配的 IP 网段。当然，我们也可以自己指定网段。只需要在创建网段时指定 `--subnet` 和 `--gateway` 参数即可。

执行命令 `docker network create --driver bridge --subnet 172.22.16.0/24 --gateway 172.22.16.1 my_net2`

![](https://sherlockblaze.com/resources/img/profession/docker/docker-network-inspect-my_net2.png)

接下来我们做实验，首先使用命令 

```sh
docker run -it --network=my_net2 --ip 172.22.16.9 busybox
docker run -it --network=my_net2 --ip 172.22.16.8 busybox
docker run -it --network=my_net busybox
```

到这里我们需要知道如下几个事情： 

1. 在前两条命令中我们指定了 `ip`，想要这样做我们需要在创建网络的时候指定 `subnet` 配置项
2. 执行上述命令行后，我们创建出三个容器，且这个状态下的网络拓扑结构如下图所示：

![](https://sherlockblaze.com/resources/img/profession/docker/docker-bridge-graph-new.png)

接下来我们讨论容器的通信问题。

## 容器间通信

**容器之间可以通过 IP、Docker DNS Server 和 joined 容器三种方式通信。**

### IP 通信

根据上段内容，我们得到了三个容器。ip地址分别是： `172.22.16.8` 、 `172.22.16.9` 以及 `172.21.0.2`。

其中，前两个属于同一个网段，后一个单独一个网段。

理论上，身处同一个网段中是可以互通的，我们在 IP 为 `172.22.16.8` 的容器中去 `ping` IP 为 `172.22.16.9` 的容器。得到的结果如下图所示：

![](https://sherlockblaze.com/resources/img/profession/docker/ping-172-22-16-9.png)

以上结果验证了我们的说法，同一网段中的容器的确和 ping 通。

接下来我们在 IP 为 `172.22.16.8` 的容器中去 `ping` IP 为 `172.21.0.2` 的容器：

![](https://sherlockblaze.com/resources/img/profession/docker/ping-172-21-0-2.png)

> 不同的网络如果加上路由应该是可以通信的。

在 Docker 宿主机上执行命令 `ip r`，我们可以得到如下结果：

![](https://sherlockblaze.com/resources/img/profession/docker/host-ip-r-router.png)

图中我们可以观察到两个网络的路由器， `172.22.16.0` 以及 `172.21.0.0`。

继续执行命令 `sysctl net.ipv4.ip_forward`，得到如图输出：

![](https://sherlockblaze.com/resources/img/profession/docker/check_ipv4_forward.png)

由此我们可以确认 ip forwarding 已启用。

到了这里我们需要查看一下问题所在，因为目前知道通过 IP 进行通信的条件是基本满足的： 1. 两个网络的路由器已经配置完毕 2. ip forwarding 已经启用。

那么是什么原因导致挂载在两个不同网桥的 busybox 无法互相访问呢？

执行命令 `sudo iptables-save` 得到以下结果，然后我们来一探究竟：

![](https://sherlockblaze.com/resources/img/profession/docker/ip-table-docker-network.png)

在图片的下方我们可以看到如下两行内容：

```
-A DOCKER-ISOLATION-STAGE-2 -o br-2d8877e1871b -j DROP
-A DOCKER-ISOLATION-STAGE-2 -o br-57b3ea2cc45d -j DROP
```

原来，iptables 把从 `br-2d8877e1871b` 和 `br-57b3ea2cc45d` 两个网桥出来的流量都给 DROP 掉了。同时，从命令规则上来看，`DOCKER-ISOLATION` 的名称，可知 docker 在设计上就是要隔离不同的 network。

那么在 Docker 中我们如何将两个网络连接起来呢？答案很简单，就是向其中一个 busybox 添加一个网卡，让它和挂载在被添加的网卡上的网络联通。

执行命令 `docker network connect my_net2 8b4a` 即可完成。

接下来我们在 ip 为 `172.21.0.2` 的 busybox 容器中执行命令 `ifconfig`，得到如下输出：

![](https://sherlockblaze.com/resources/img/profession/docker/ifconfig-after-connect-network.png)

我们可以清楚的观察到多了一个 ip 为 `172.22.16.2` 的网卡 `eth1`。此时，网络拓扑图如下所示：

![](https://sherlockblaze.com/resources/img/profession/docker/docker-bridge-graph-after-connect.png)

致此，我们就可以用新 IP 让三个 busybox 实现互通了。