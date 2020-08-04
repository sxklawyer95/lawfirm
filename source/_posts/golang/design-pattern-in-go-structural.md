---
title: Design Pattern in Go (Structural Pattern)
tags:
  - Golang
date: 2019-12-06
---

## Composite design pattern

**The Composite design pattern favors composition over inheritance.** All in all, Go doesn't have inheritance because it doesn't need it!

- **composite_test.go**

```golang
package composition

import "testing"

func TestAthlete_Train(t *testing.T) {
    athlete := Athlete{}
    athlete.Train()
}

func TestSwimmer_Swim(t *testing.T) {
    localSwim := Swim
    swimmer := CompositeSwimmerA{
        MySwim: &localSwim,
    }
    swimmer.MyAthlete.Train()
    (*swimmer.MySwim)()
}

func TestAnimal_Swim(t *testing.T) {
    fish := Shark{
        Swim: Swim,
    }

    fish.Eat()
    fish.Swim()
}

func TestSwimmer_Swim2(t *testing.T) {
    swimmer := CompositeSwimmerB{
        &Athlete{},
        &SwimmerImplementor{},
    }

    swimmer.Train()
    swimmer.Swim()
}

func TestTree(t *testing.T) {
    root := Tree{
        LeafValue: 0,
        Right: &Tree{
            LeafValue: 5,
            Right:     &Tree{6, nil, nil},
        },
        Left: &Tree{4, nil, nil},
    }

    println(root.Right.Right.LeafValue)
}

func TestSon_GetParentField(t *testing.T) {
    son := Son{}
    GetParentField(son.P)
}
```

- **composite.go**

```golang
package composition

type Athlete struct{}

func (a *Athlete) Train() {
    println("Training")
}

func Swim() {
    println("Swimming!")
}

type CompositeSwimmerA struct {
    MyAthlete Athlete
    MySwim    *func()
}

type Trainer interface {
    Train()
}

type Swimmer interface {
    Swim()
}

type SwimmerImplementor struct{}

func (s *SwimmerImplementor) Swim() {
    println("Swimming")
}

type CompositeSwimmerB struct {
    Trainer
    Swimmer
}

type Animal struct{}

func (r *Animal) Eat() {
    println("Eating")
}

type Shark struct {
    Animal
    Swim func()
}

type Tree struct {
    LeafValue int
    Right     *Tree
    Left      *Tree
}

type Parent struct {
    SomeField int
}

type Son struct {
    P Parent
}

func GetParentField(p Parent) int {
    return p.SomeField
}
```

At this point, you should be really comfortable using the Composite design pattern. It's very idiomatic Go feature. The Composite design pattern makes our structures predictable but also allows us to create most of the design patterns.

## Adapter design pattern

**In Go, an adapter will allow us to use something that wasn't built for a specific task at the beginning.**

The Adapter pattern is very useful when, for example, an interface gets outdated and it's not possible to replace it easily or fast. Instead, you create a new interface to deal with the current needs of your application, which, under the hood, uses implementations of the old interface.

> Just keep in mind that extensibility in code is only possible through the use of design patterns and interface-oriented programming.

### Objectives

The Adapter design pattern will help you fit the needs of two parts of the code that are incompatible at first.

```golang
package structural

import "fmt"

type LegacyPrinter interface {
    Print(s string) string
}

type MyLegacyPrinter struct {}

func (l *MyLegacyPrinter) Print(s string) (newMsg string) {
    newMsg = fmt.Sprintf("Legacy Printer: %s\n", s)
    println(newMsg)
    return
}

type NewPrinter interface {
    PrintStored() string
}

type PrinterAdapter struct {
    oldPrinter LegacyPrinter
    Msg        string
}

func (p *PrinterAdapter) PrintStored() (newMsg string) {
    if p.OldPrinter != nil {
        newMsg = fmt.Sprintf("Adapter: %s", p.Msg)
        newMsg = p.OldPrinter.Print(newMsg)
    } else {
        newMsg = p.Msg
    }

    return
}
```

## Bridge design pattern

The Bridge pattern is a design with a slightly cryptic definition from the original Gang of Four book. **It decouples an abstraction from its implementation so that the two can vary independently.** The cryptic explanation just means that you could even decouple the most basic form of functionality: **decouple an object from what it does.**

The Bridge pattern tries to decouple things as usual with design patterns. It decouples abstraction (an object) from its implementation (the thing that the object does). This way, we can change what an object does as much as we want. It also allows us to change the abstracted object while reusing the same implementation.

The objective of the Bride pattern is to bring flexibility to a struct that change often. Knowing the inputs and outputs of a method, it allows us to change code without knowing too much about it and leaving the freedom for both sides to be modified more easily.

```golang
package structural

import (
    "errors"
    "fmt"
    "io"
)

type PrinterAPI interface {
    PrintMessage(string) error
}

type PrinterAPI1 struct {}

func (d *PrinterAPI1) PrintMessage(msg string) error {
    fmt.Printf("%s\n", msg)
    return nil
}

type PrinterAPI2 struct{}

func (d *PrinterAPI2) PrintMessage(msg string) error {
    if d.Writer == nil {
        return errors.New("You need to pass an io.Writer to PrinterAPI2")
    }
    fmt.Printf(d.Writer, "%s", msg)
    return nil
}

type PrinterAbstraction interface {
    Print() error
}

type NormalPrinter struct {
    Msg     string
    Printer PrinterAPI
}

func (c *NormalPrinter) Print() error {
    c.Printer.PrintMessage(c.Msg)
    return nil
}

type PacktPrinter struct {
    Msg     string
    Printer PrinterAPI
}

func (c *PacktPrinter) Print() error {
    c.Printer.PrintMessage(fmt.Sprintf("Message from Packt: %s", c.Msg))
    return nil
}
```

## Proxy Design Pattern

The Proxy Pattern usually wraps an object to hide some of its characteristic. These characteristics could be the fact that it is a remote object(remote proxy), a very heavy object such as a very big image or the dump of a terabyte database(virtual proxy), or a restricted access object(protection proxy).

### Objectives

The possibilities of the Proxy pattern are many, but in general, they all try to provide the same following functionalities:

- Hide an object behind the proxy so the features can be hidden, restricted, and so on
- Provide a new abstraction layer that is easy to work with, and can be changed easily

```golang
package proxy

import "fmt"

type User struct {
    ID int32
}

type UserFinder interface {
    FindUser(id string) (User, error)
}

type UserList []User

func (t *UserList) FindUser(id int32) (User, error) {
    for i := 0; i < len(*t); i++ {
        if (*t)[i].ID) == id {
            return (*t)[i], nil
        }
    }

    return User{}, fmt.Errorf("User %s could not be found\n", id)
}

func (t *UserList) addUser(newUser User) {
    *t = append(*t, newUser)
}

type UserListProxy struct {
    MockedDatabase      *UserList
    StackCache          UserList
    StatckSize          int
    LastSearchUsedCache bool
}

func (u *UserListProxy) addUserToStack(user User) {
    if len(u.StackCache) >= u.StatckSize {
        u.StackCache = append(u.StackCache[1:], user)
    } else {
        u.StackCache.addUser(user)
    }
}

func (u *UserListProxy) FindUser(id int32) (User, error) {
    user, err := u.StackCache.FindUser(id)
    if err == nil {
        fmt.Println("Returning user from cache")
        u.LastSearchUsedCache = true
        return user, nil
    }

    user, err = u.MockedDatabase.FindUser(id)
    if err != nil {
        return User{}, err
    }

    u.addUserToStack(user)

    fmt.Println("Returning user from database")
    u.LastSearchUsedCache = false
    return user, nil
}
```

## Decorator Design Pattern

**The Decorator design pattern allows you to decorate an already existing type with more functional features without actually touching it.** It uses an approach similar to ***matryoshka dolls***, where you have a small doll that you can put inside a doll of the same shape but bigger, and so on and so forth.

The Decorator type implements the same interface of the type it decorates, and stores an instance of that type in its members.

This way, you can stack as many decorators (dolls) as you want by simply storing the old decorator in a field of the new one.

### Objectives

When you think about extending legacy code without the risk of breaking something, you should think of the Decorator pattern first.

So, precisely when are we going to use the Decorator pattern?

- When you need to add functionality to some code that you don't have access to, or you don't want to modify to avoid a negative effect on the code, and follow the open/colse principle (like legacy code)
- When you want the functionality of an object to be created or altered dynamically, and the number of features is unknown and could grow fast

- **pizza_decorator.go**

```golang
package decorator

import (
    "errors"
    "fmt"
)

type IngredientAdder interface {
    AddIngredient() (string, error)
}

type PizzaDecorator struct {
    Ingredient IngredientAdder
}

func (p *PizzaDecorator) AddIngredient() (string, error) {
    return "Pizza with the following ingredients:", nil
}

type Meat struct {
    Ingredient IngredientAdder
}

func (m *Meat) AddIngredient() (string, error) {
    if m.Ingredient == nil {
        return "", errors.New("An IngredientAdder is needed on the Ingredient field of the Meat")
    }

    s, err := m.Ingredient.AddIngredient()
    if err != nil {
        return "", err
    }
    return fmt.Sprintf("%s %s,", s, "meat"), nil
}

type Onion struct {
    Ingredient IngredientAdder
}

func (o *Onion) AddIngredient() (string, error) {
    if o.Ingredient == nil {
        return "", errors.New("An ingredientAdder is needed on the Ingredient field of the Onion")
    }
    s, err := o.Ingredient.AddIngredient()
    if err != nil {
        return "", err
    }
    return fmt.Sprintf("%s %s,", s, "onion"), nil
}
```

- **server_decorator.go**

```golang
package main

import (
    "fmt"
    "io"
    "log"
    "net/http"
    "os"
)

type MyServer struct{}

func (m *MyServer) ServeHTTP(w http.ResponseWriter, r *http.request) {
    fmt.Fprintln(w, "Hello Decorator!")
}

type LoggerMiddleware struct {
    Handler   http.Handler
    LogWriter io.Writer
}

func (l *LoggerMiddleware) ServeHTTP(w http.ResponseWriter, r *http.request) {
    fmt.Fprintf(l.LogWriter, "Request URI: %s\n", r.RequestURI)
    fmt.Fprintf(l.LogWriter, "Host: %s\n", r.Host)
    fmt.Fprintf(l.LogWirter, "Content Length: %d\n", r.ContentLength)
    fmt.Fprintf(l.LogWriter, "Method: %s\n", r.Method)
    fmt.Fprintf(l.LogWriter, "--------------------\n")
    l.Handler.ServeHttp(w, r)
}

type SimpleAuthMiddleware struct {
    Handler  http.Handler
    User     string
    Password string
}

func (s *SimpleAuthMiddleware) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    user, pass, ok := r.BasicAuth()

    if ok {
        if user == s.User && pass == s.Password {
            s.Handler.ServeHTTP(w, r)
        } else {
            fmt.Fprintf(w, "User or Password incorrect\n")
        }
    } else {
        fmt.Fprintln(w, "Error trying to retrieve data from Basic auth")
    }
}

func main() {
    fmt.Println("Enter the type number of server you want to launch from the" + " following:")
    fmt.Println("1.- Plain server")
    fmt.Println("2.- Server with logging")
    fmt.Println("3.- Server with logging and authentication")

    var selection int
    fmt.Fscanf(os.Stdin, "%d", &selection)

    var mySuperServer http.Handler

    switch selection {
    case 1:
        mySuperServer = new(MyServer)
    case 2:
        mySuperServer = &LoggerMiddleware{
            Handler:   new(MyServer),
            LogWriter: os.Stdout,
        }
    case 3:
        var user, password string
        fmt.Println("Enter user and password separated by a space")
        fmt.Fscanf(os.Stdin, "%s %s", &user, &password)

        mySuperServer = &LoggerMiddleware{
            Handler: &SimpleAuthMiddleware{
                Handler:  new(MyServer),
                User:     user,
                Password: password,
            },
            LogWriter: os.Stdout,
        }
    default:
        mySuperServer = new(MyServer)
    }

    http.Handle("/", mySuperServer)

    log.Fatal(http.ListenAndServe(:8080, nil))
}
```

In the Decorator pattern, we decorate a type dynamically. This means that the decoration may or may not be there, or it may be composed of one or many types. If you remember, the Proxy pattern wraps a type in a similar fashion, but it does so at compile time and it's more like a way to access some type.

At the same time, a decorator might implement the entire interface that the type it decorates also implements or not. So you can have an interface with 10 methods and a decorator that just implements one of them and it will still be valid. This is a very powerful feature but also very prone to undesired behaviors at runtime if you forget to implement any interface method.

In this aspect, you may think that the Proxy pattern is less flexible, and it is. But the Decorator pattern is weaker, as you could have errors at runtime, which you can avoid at compile time by using the Proxy pattern. **Just keep in mind that the Decorator is commonly used when you want to add functionality to an object at runtime, like in our web server.** It's a compromise between what you need and what you want to sacrifice to achieve it.

## Facade Design Pattern

**Proxy pattern was a way to wrap an type to hide some of its features of complexity from the user.**

**Imagine that we group many proxies in a single point such as a file or a library.** This could be Facade patter.

A facade, in architectural terms, is the front wall that hides the rooms and corridors of a building. It protects its inhabitants from code and rain, and provides them privacy. It orders and divides the dwellings.

The Facade design pattern shields the code from unwanted access, orders some calls, and hides the complexity scope from the user.

### Objectives

You use Facade when you want to hide the complexity of some tasks, especially when most of them share utilities(such as authentication in an API). A library is a form of facade, where someone has to provide some methods for a developer to do certain things in a friendly way. This way, if a developer needs to use your library, he doesn't need to know all the inner tasks to retrieve the result he/she wants.

- When you want to decrease the complexity of some parts of our code. You hide that complexity behind the facade by providing a more easy-to-use method.
- When you want to group actions that are cross-related in a single place.
- When you want to build a library so that others can use your products without worrying about how it all works.

```golang
package openWeatherMap

import (
    "encoding/json"
    "fmt"
    "io"
    "io/ioutil"
    "net/http"
)

type CurrentWeatherDataRetriever interface {
    GetByGeoCoordinates(lat, lon float32) (*Weather, error)
    GetByCityAndCountryCode(city, countryCode string) (*Weather, error)
}

type CurrentWeatherData struct {
    APIkey string
}

type Weather struct {
    Coord struct {
        Lon float32 `json:"lon"`
        Lat float32 `json:"lat"`
    } `json:"coord"`
    Weather []struct {
        Id          int    `json:"id"`
        Main        string `json:"main"`
        Description string `json:"description"`
        Icon        string `json:"icon"`
    } `json:"weather"`
    Base string `json:"base"`
    Main struct {
        Temp     float32 `json:"temp"`
        Pressure float32 `json:"pressure"`
        Humidity float32 `json:"humidity"`
        TempMin  float32 `json:"temp_min"`
        TempMax  float32 `json:"temp_max"`
    } `json:"main"`
    Wind struct {
        Speed float32 `json:"speed"`
        Deg   float32 `json:"deg"`
    } `json:"wind"`
    Clouds struct {
        All int `json:"all"`
    } `json:"clouds"`
    Rain struct {
        ThreeHours float32 `json:"3h"`
    } `json:"rain"`
    Dt  uint32 `json:"dt"`
    Sys struct {
        Type    int     `json:"type"`
        ID      int     `json:"id"`
        Message float32 `json:"message"`
        Country string  `json:"country"`
        Sunrise int     `json:"sunrise"`
        Sunset  int     `json:"sunset"`
    } `json:"sys"`
    ID   int    `json:"id"`
    Name string `json:"name"`
    Cod  int    `json:"cod"`
}

const (
    commonRequestPrefix = "http://api.openweathermap.org/data/2.5/"
    weatherByCityName = commonRequestPrefix + "weather?q=%s,%s&APPID=%s"
    weatherByGeographicalCoordinates = commonRequestPrefix + "weather?lat=%f&lon=%f&APPID=%s"
)

func (c *CurrentWeatherData) GetByGeoCoordinates(lat, lon float32) (weather *Weather, err error) {
    return c.doRequest(fmt.Sprintf(weatherByGeographicalCoordinates, lat, lon, c.APIkey))
}

func (c *CurrentWeatherData) GetByCityAndCountryCode(city, countryCode string) (weather *Weather, err error) {
    return c.doRequest(fmt.Sprintf(weatherByCityName, city, countryCode, c.APIkey))
}

func (c *CurrentWeatherData) responseParser(body io.Reader) (*Weather, error) {
    w := new(Weather)
    err := json.NewDecoder(body).Decode(w)
    if err != nil {
        return nil, err
    }

    return w, nil
}

func (o *CurrentWeatherData) doRequest(uri string) (weather *Weather, err error) {
    client := &http.Client{}
    req, err := http.NewRequest("GET", uri, nil)
    if err != nil {
        return
    }
    req.Header.Set("Content-Type", "application/json")

    resp, err := client.Do(req)
    if err != nil {
        return
    }

    if resp.StatusCode != 200 {
        byt, errMsg := ioutil.ReadAll(resp.Body)
        defer resp.Body.Close()
        if errMsg == nil {
            errMsg = fmt.Errorf("%s", string(byt))
        }
        err = fmt.Errorf("Status code was %d, aborting. Error message was:\n%s\n", resp.StatusCode, errMsg)

        return
    }

    weather, err = o.responseParser(resp.Body)
    resp.Body.Close()

    return
}
```

## Flyweight Design Pattern

Flyweight design pattern is very commonly used in computer graphics and the video game industry, but not so much in enterprise applications.

**Flyweight is a pattern which allows sharing the state of a heavy object between many instance of some type.**

Imagine that you have to create and store too many objects of some heavy type that are fundamentally equal. You'll run out of memory pretty quickly.

This problem can be easily solved with the Flyweight pattern, **with additional help of the Factory pattern.**

### Objectives

Thanks to the Flyweight pattern, we can share all possible states of objects in a single common object, and thus minimize object creation by using pointers to already created objects.

> To give an example, we are going to simulate something that you find on betting webpages. Imagine the final match of the European championship, which is viewed by millions of people across the continent.
Now imagine that we own a betting webpage, where we provide historical information about every team in Europe. This is plenty of information, which is usually stored in some distributed database, and each team has, literally, megabytes of information about their players, matches, championships, and so on.
If a million users access information about a team and a new instance of the information is created for each user querying for historical data, we will run out of memory in the blink of an eye. With the Proxy solution, we could make a cache of the ***n*** most recent searches to speed up queries, but if we return a clone for every team, we will still get short on memory(but faster thanks to our cache).
Instead, we will store each team's information just once, and we will deliver references to them to the users. So, if we face a million users trying to access information about a match, we will actually just have two teams in memory with a million pointers to the same memory direction.

- **flyweight.go**

```golang
package flyweight

import "time"

const (
    TEAM_A = iota
    TEAM_B
)

type Player struct {
    Name         string
    Surname      string
    PreviousTeam uint64
    Photo        []byte
}

type HistoricalData struct {
    Year          uint8
    LeagueResults []Match
}

type Team struct {
    ID             uint64
    Name           string
    Shield         []byte
    Players        []Player
    HistoricalData []HistoricalData
}

type Match struct {
    Date          time.Time
    VisitorID     uint64
    LocalID       uint64
    LocalScore    byte
    VisitorScore  byte
    LocalShoots   uint16
    VisitorShoots uint16
}

func getTeamFactory(team int) Team {
    switch team {
    case TEAM_B:
        return TEAM{
            ID:   2,
            Name: "TEAM_B",
        }
    default:
        return Team{
            ID:   1,
            Name: "TEAM_A",
        }
    }
}

func NewTeamFactory() teamFlyweightFactory {
    return teamFlyweightFactory{
        createdTeams: make(map[int]*Team, 0),
    }
}

type teamFlyweightFactory struct {
    createdTeams map[int]*Team
}

func (t *teamFlyweightFactory) GetTeam(teamName int) *Team {
    if t.createdTeams[teamName] != nil {
        return t.createdTeams[teamName]
    }

    team := getTeamFactory(teamName)
    t.createdTeams[teamName] = &team

    return t.createdTeams[teamName]
}

func (t *teamFlyweightFactory) GetNumberOfObjects() int {
    return len(t.createdTeams)
}
```

- **flyweight_test.go**

```golang
package flyweight

import (
    "fmt"
    "testing"
)

func TestTeamFlyweightFactory_GetTeam(t *testing.T) {
    factory := NewTeamFactory()

    teamA1 := factory.GetTeam(TEAM_A)
    if teamA1 == nil {
        t.Error("The pointer to the TEAM_A was nil")
    }

    teamA2 := factory.GetTeam(TEAM_A)
    if teamA2 == nil {
        t.Error("The pointer to the TEAM_A was nil")
    }

    if teamA1 != teamA2 {
        t.Error("TEAM_A objects weren't the same")
    }

    if factory.GetNumberOfObjects() != 1 {
        t.Errorf("The number of objects created was not 1: %d\n", factory.GetNumberOfObjects())
    }
}

func Test_HighVolume(t *testing.T) {
    factory := NewTeamFactory()

    teams := make([]*Team, 500000*2)
    for i := 0; i < 500000; i++ {
        teams[i] = factory.GetTeam(TEAM_A)
    }

    for i := 500000; i < 2*500000; i++ {
        teams[i] = factory.GetTeam(TEAM_B)
    }

    if factory.GetNumberOfObjects() != 2 {
        t.Errorf("The number of objects created was not 2: %d\n", factory.GetNumberOfObjects())
    }

    for i := 0; i < 3; i++ {
        fmt.Printf("Pointer %d points to %p and is located in %p\n", i, teams[i], &teams[i])
    }
}
```

### What's the difference between Singleton and Flyweight?

With the Singleton pattern, we ensure that the same type is created only once. Also, the Singleton pattern is a Creational pattern. With Flyweight, which is a Structural pattern, **we aren't worried about how the objects are created, but about how to structure a type to contain heavy information in a light way.**