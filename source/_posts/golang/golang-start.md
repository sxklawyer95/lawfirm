---
title: Golang 初探
tags:
  - Golang
date: 2019-05-07
---

> 本学习笔记总结自： [《Go in Action》](https://www.amazon.com/Go-Action-William-Kennedy/dp/1617291781/ref=sr_1_1?keywords=go+in+action&qid=1557230058&s=gateway&sr=8-1)

## 优势

1. Golang 是一种让代码分享更加容易的编程语言
2. 提供了更高效的代码复用手段
3. 高效利用服务器上的所有核心
4. 编译速度很快，即使是大型项目
5. 开发速度快
6. 现代化的内存管理机制

## 类型系统

1. 类型简单。类型之间更多使用了组合的方式，而非继承
2. 利用接口对一组行为建模。利用了鸭子类型，即一个东西看起来是只鸭子，那它就是只鸭子

## Hello World

```golang
package main

import "fmt" // 引入外部包

func main () { // 主方法，程序的入口
    fmt.Println("Hello World!")
}
```

## More than Hello-World

### 功能描述

从不同的数据源拉取数据，将数据内容与一组搜索项做对比，然后将匹配的内容显示在终端窗口。这个程序会读取文本文件，进行网络调用，解码 XML 和 JSON 成为结构化类型数据，并且利用 Go 语言的并发机制保证这些操作的速度。

### 架构图

![](https://sherlockblaze.com/resources/img/code/golang/architecture-first-go.png)

### 简单解说

我们知道，任何程序主要做的无非三件事，输入，处理，输出。

- 输入: 那一组等待搜索的数据源，我们需要有一个方法去获取
- 处理: 也就是我们执行搜索的部分，我们通过启动几个 goroutine(协程)，执行既定好的代码去完成
- 输出: 主要也就是显示结果那一部分

### 项目结构

```
code
│   main --- 主程序
│
└───data
│   │   data.json --- 数据源
│   
└───search
│   │   default.go --- 搜索数据用的默认匹配器
│   │   feed.go    --- 用于读取 json 数据文件
│   |   match.go   --- 用于支持不同匹配器的接口
│   |   search.go  --- 执行搜索的主控制逻辑
│    
└───matchers
    │   rss.go     --- 搜索 rss 源的匹配器
```

### 代码实现及分析

按照输入、输出、处理的顺序来遍历一遍代码。

#### 输入

已知 `feed.go` 用于读取 json 的数据文件，所以我们从 `feed.go` 的代码开始编写。

```golang
package search

import (
    "encoding/json"
    "os"
)

const dataFile = "data/data.json"

// 定义结构体
type Feed struct {
    Name string `json:"site"` // 结构体中的名称为: Name，对应 json 中的标签为 site
    URI  string `json:"line"`
    Type string `json:"type"`
}

func RetrieveFeeds() ([]*Feed, error) {
    // 打开文件
    file, err := os.Open(dataFile)
    if err != nil {
        return nil, err
    }
    
    // defer 关键字标注的语句会在方法结束时调用
    defer file.Close()
  
    // Feed 指针切片  
    var feeds []*Feed
    // 利用 json 库，解析 json 中的内容，并且放到 feeds 中
    err = json.NewDecoder(file).Decode(&feeds)
    
    return feeds, err
}
```

#### 输出

我们来看看 `match.go` 文件中做了什么

```golang
package search

import (
    "log"
)

// 结构体 Result
type Result struct {
    Filed   string
    Content string
}

// 定义一个 Matcher 接口
type Matcher interface {
    Search(feed *Feed, searchTerm string) ([]*Result, error)
}

// Match 方法
func Match(matcher Matcher, feed *Feed, searchTerm string, results chan<- *Result) {
    // 调用
    searchResults, err := matcher.Search(feed, searchTerm)
    if err != nil {
        log.Println(err)
        return
    }

    for _, result := range searchResults {
        results <- result
    }
}

func Display(results chan *Result) {
    for result := range results {
        log.Printf("%s:\n%s\n\n", result.Field, result.Content)
    }
}
```

#### 处理

获取到我们输入数据后，我们就要开始处理他们了。

首先我们在 `default.go` 文件里声明一个默认解析器，当然，do nothing

```golang
package search

type defaultMatcher struct{}

func init() {
    var matcher defaultMatcher
    // 注册 default matcher
    Register("default", matcher)
}

// 实现 Search 接口，Golang 实现接口的方式很简单高效，直接针对某个类型，实现对应的方法即可
func (m defaultMatcher) Search(feed *Feed, searchTerm string) ([]*Result, error) {
    return nil, nil
}
```

接下来我们来看 `search.go` 文件

```golang
package search

import (
    "log"
    "sync"
)

// 声明一个 map，由 matcher 名称(键) + 实际的 Mathcer(值) 组成
var matchers = make(map[string]Matcher)

func Run(searchTerm string) {
    // 调用 RetrieveFeeds 方法来获取 data.json 里的内容
    feeds, err := RetrieveFeeds()
    if err != nil {
        log.Fatal(err)
    }

    // 创建通道
    results := make(chan *Result)

    var waitGroup sync.WaitGroup

    // 声明 waitGroup 并添加 feeds 长度
    waitGroup.Add(len(feeds))

    for _, feed := range feeds {
        // 获取 matcher
        matcher, exists := matchers[feed.Type]
        if !exists {
            matcher = matchers["default"]
        }

        // 执行协程，开启 matcher 之旅
        go func(matcher Matcher, feed *Feed) {
            Match(matcher, feed, searchTerm, results)
            waitGroup.Done()
        }(matcher, feed) // 括号里的内容是传入的参数
    }

    // 跟踪结果的 goroutine
    go func() {
        waitGroup.Wait()
        close(results)
    }()

    Display(results)
}

// 注册 matcher
func Register(feedType string, matcher Mathcer) {
    if _, exists := matchers[feedType]; exists {
        log.Fatalln(feedType, "Matcher already Registered")
    }

    log.Println("Register", feedType, "matcher")
    matchers[feedType] = matcher
}
```

重头戏来了，之前的工作主要就是通过找到 matcher，调用 matcher 中的方法来执行数据的处理

```golang
package matchers

import (
    "encoding/xml"
    "errors"
    "fmt"
    "log"
    "net/http"
    "regexp"

    "search"
)

// 声明一些结构体去匹配 xml 中的内容，你可以仅使用一个 type
type (
    item struct {
        XMLName     xml.Name `xml:"item"`
        PubDate     string   `xml:"pubDate"`
        Title       string   `xml:"title"`
        Description string   `xml:"description"`
        Link        string   `xml:"link"`
        GUID        string   `xml:"guid"`
        GeoRssPoint string   `xml:"georss:point"`
    }

    image struct {
        XMLNAME xml.Name `xml:"image"`
        URL     string   `xml:"url"`
        Title   string   `xml:"title"`
        Link    string   `xml:"link"`
    }

    channel struct {
        XMLNAME        xml.Name `xml:"channel"`
        Title          string   `xml:"title"`
        Description    string   `xml:"description"`
        Link           string   `xml:"link"`
        PubDate        string   `xml:"pubDate"`
        LastBuildDate  string   `xml:"lastBuildDate"`
        TTL            string   `xml:"ttl"`
        Language       string   `xml:"language"`
        ManagingEditor string   `xml:"managingEditor"`
        WebMaster      string   `xml:"webMaster"`
        Image          image    `xml:"image"`
        Item           []item   `xml:"item"`
    }
)

type rssMatcher struct {}

// 初始化，注册 Matcher
func init() {
    var matcher rssMatcher
    search.Register("rss", matcher)
}

// 为 rssMatcher 实现 Search 方法
func (m rssMatcher) Search(feed *search.Feed, searchTerm string) ([]*search.Result, error) {
    var results []*result.Result
    log.Printf("Search Feed Type[%s] Site[%s] For URI[%s]\n", feed.Type, feed.Name, feed.URI)

    // 获取 document
    document, err := m.retrieve(feed)
    if err != nil {
        return nil, err
    }

    // 进行匹配
    for _, channelItem := range document.Channel.Item {
        matched, err := regexp.MatchString(searchTerm, channelItem.Title)
        if err != nil {
            return nil, err
        }

        if matched {
            results = append(results, &search.Result {
                Field: "Title",
                Content: channelItem.Title,
            })
        }
    }
        return results, nil
}

// 获取 feed 流
func (m rssMatcher) retrieve(feed *search.Feed) (*rssDocument, error) {
    if feed.URI == "" {
        return nil, errors.New("No rss feed url provided")
    }

    // 获取相应，并解码
    resp, err := http.Get(feed.URI)
    if err != nil {
        return nil, err
    }

    defer resp.Body.Close()

    if resp.StatusCode != 200 {
        return nil, fmt.Errorf("HTTP Response Error %d\n", resp.StatusCode)
    }

    var document rssDocument
    err = xml.NewDecoder(resp.Body).Decode(&document)
    return &document, err
}
```

#### 主程序

接下来我们需要一个主程序来作为整个程序的入口。

```golang
package main

import (
    "log"
    "os"

    _ "matchers"
    "search"
)

func init() {
    log.SetOutput(os.Stdout)
}

func main() {
    search.Run("president")
}
```

你可以在[这里](https://sherlockblaze.com/resources/file/code/golang/data.json)下载 `data.json`。