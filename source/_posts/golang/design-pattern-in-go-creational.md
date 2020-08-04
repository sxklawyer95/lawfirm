---
title: Design Pattern in Go (Creational Pattern)
tags:
  - Golang
date: 2019-12-06
---

## The Singleton Pattern

It will provide you with a single instance of an object, and guarantee that there are no duplicates.

At the first call to use the instance, it's created and then reused between all the parts in the application that need to use that particular behavior.

You'll use the Singleton pattern in many different situations. For example:

- When you want to use the same connection to a database to make every query
- When you open a **Secure Shell(SSh)** connection to a server to do a few tasks, and don't want to reopen the connection for each task
- If you need to limit the access to some variable or space, you use a Singleton as the door to this variable
- If you need to limit the number of calls to some places, you create a Singleton instance to make the calls in the accepted window

The possibilities are endless, and we have just mentioned some of them.

### Objectives

As a general guide, we consider using the Singleton pattern when the following rule applies:

- We need a single, shared value, of some particular type.
- We need to restrict object creation of some type to a single unit along the entire program.

```golang
package creational

type singleton struct {
    count int
}

var instance *singleton

func GetInstance() *singleton {
    if install == nil {
        instance = new(singleton)
    }

    return instance
}

func (s *singleton) AddOne() int {
    s.count++
    return s.count
}

func (s *singleton) GetCount() int {
    return s.count
}
```

> Just keep in mind that the Singleton pattern will give you the power to have a unique instance of some struct in your application and that no package can create any clone of this struct.
With Singleton, you are also hiding the complexity of creating the object, in case it requires some computation, and the pitfall of creating it every time you need an instance of it if all of them are similar. All this code writing, checking if the variable already exists, and storage, are encapsulated in the singleton in the singleton and you won't need to repeat it everywhere if you use a global variable.
**The implementation up there is not thread safe**

## The Builder Design Pattern

**Reusing an algorithm to create many implementations of an interface.**

The Builder Pattern helps us construct complex objects without directly instantiating their struct, or writing the logic they require. Imagine an object that could have dozens of fields that are more complex structs themselves. Now Imagine that you have many objects with these characteristics, and you could have more. We don't want to write the logic to create all these objects in the package that just needs to use the objects.

You could also have an object that is composed of many objects, something that's really idiomatic in Go, as it doesn't support inheritance.

At the same time, you could be using the same technique to create many types of objects. For example, you'll use almost the same technique to build a car as you would build a bus, except that they'll be different sizes and number of seats, so **why don't we reuse the construction process?**

### Objectives

A Builder design pattern tries to:

- Abstract complex creations so that object creation is separated from the object user
- Create an object step by step by filling its fields and creating the embedded objects
- Reuse the object creation algorithm between many objects

```golang
package creational

type BuildProcess interface {
    SetWheels() BuildProcess
    SetSeats() BuildProcess
    SetStructure() BuildProcess
    GetVehicle() VehicleProduct
}

// Director
type MenufacturingDirector struct {
    builder BuildProcess
}

func (f *ManufacturingDirector) Construct() {
    f.builder.SetSeats().SetStructure().SetWheels()
}

func (f *ManufacturingDirector) SetBuilder() {
    f.builder = b
}

// Product
type VehicleProduct struct {
    Wheels    int
    Seats     int
    Structure string
}

// A Builder of type car
type CarBuilder struct {
    v VehicleProduct
}

func (c *CarBuilder) SetWheels() BuildProcess {
    c.v.Wheels = 4
    return c
}

func (c *CarBuilder) SetSeats() BuildProcess {
    c.v.Seats = 5
    return c
}

func (c *CarBuilder) SetStructure() BuildProcess {
    c.v.Structure = "Car"
}

func (c *CarBuilder) GetVehicle() VehicleProduct {
    return c.v
}

// A builder of type motorbike
type BikeBuilder struct {
    v VehicleProduct
}

func (b *BikeBuilder) SetWheels() BuildProcess {
    b.v.Wheels = 2
    return b
}

func (b *BikeBuilder) SetSeats() BuildProcess {
    b.v.Seats = 2
    return b
}

func (b *BikeBuilder) SetStructure() BuildProcess {
    b.v.Structure = "motorbike"
    return b
}

func (b *BikeBuilder) GetVehicle() VehicleProduct {
    return b.v
}

// A builder of type bus
type BusBuilder struct {
    v VehicleProduct
}

func (b *BusBuilder) SetWheels() BuildProcess {
    b.v.Wheels = 8
    return b
}

func (b *BusBuilder) SetSeats() BuildProcess {
    b.v.Seats = 30
    return b
}

func (b *BusBuilder) SetStructure() BuildProcess {
    return b
}

func (b *BusBuilder) GetVehicle() VehicleProduct {
    return b.v
}
```

### Wrapping up the Builder design pattern

The Builder design pattern helps us maintain an unpredictable number of products by using a common construction algorithm that is used by the director. The construction process is always abstracted from the user of the product.

At the same time, having a defined construction pattern helps when a newcomer to our source code needs to add a new product to the pipeline.

## Factory method

**delegating the creation of different types of payment**

Its purpose is to abstract the user from the knowledge of the struct he needs to achieve for a specific purpose, such as retrieving some value, maybe from a web service or database.
The user only needs an interface that provides him this value. By delegating this decision to a Factory, this Factory can provide an interface that fits the user needs.

When using the Factory method design pattern, we gain an extra layer of encapsulation so that our program can grow in a controlled environment. With the Factory method, we delegate the creation of families of objects to a different package or object to abstract us from the knowledge of the pool of possible objects we could use.

### Objectives

After the previous description, the following objectives of the Factory Method design pattern must be clear to you:

- Delegating the creation of new instances of structures to a different part of the program
- Working at the interface level instead of with concrete implementations
- Grouping families of objects to obtain a family object creator

```golang
package creational

import (
    "errors"
    "fmt"
)

type PaymentMethod interface {
    Pay(amount float32) string
}

const (
    Cash      = 1
    DebitCard = 2
)

func GetPaymentMethod(m int) (PaymentMethod, error) {
    switch m {
    case Cash:
        return new(CashPM), nil
    case DebitCard:
        return new(NewDebitCardPM), nil
    default:
        return nil, errors.New(fmt.Sprintf("Payment method %d not recognized\n"), m)
    }
}

type CashPM struct {}
type DebitCardPM struct{}

func (c *CashPM) Pay(amount float32) string {
    return fmt.Sprintf("%0.2f payed using cash\n", amount)
}

func (c *DebitCardPM) Pay(amount float32) string {
    return fmt.Sprintf("%0.2f payed using debit card\n", amount)
}

type NewDebitCardPM struct{}

func (d *NewDebitCardPM) Pay(amount float32) string {
    return fmt.Sprintf("%0.2f payed using debit card(new)\n", amount)
}
```

## Abstract Factory

**A factory of factories.**

The Abstract Factory design pattern is a new layer of grouping to achieve a bigger(and more complex) composite object, which is used through its interfaces. 
The idea behind grouping objects in families and grouping families is to have big factories that can be interchangeable and can grow more easily. **In the early stages of development, it is also easier to work with factories and abstract factories than to wait until all concrete implementations are done to start your code.**

### Objectives

Grouping related families of objects is very convenient when your object number is growing so much that creating a unique point to get them all seems the only way to gain the flexibility of the runtime object creation.

- Provide a new layer of encapsulation for Factory methods that return a common interface for all factories
- Group common factories into a super Factory (also called a factory of factories)

- **vehicle_factory.go**

```golang
package abstract_factory

import (
    "errors"
    "fmt"
)

type VehicleFactory interface {
    GetVehicle(v int) (Vehicle, error)
}

const (
    CarFactoryType       = 1
    MotorbikeFactoryType = 2
)

func GetVehicleFactory(f int) (VehicleFactory, error) {
    switch f {
    case CarFactoryType:
        return new(CarFactoryType), nil
    case MotorbikeFactoryType:
        return new(MotorbikeFactoryType), nil
    default:
        return nil, errors.New(fmt.Sprintf("Factory with id %d not recognized\n", f))
    }
}
```

- **car_factory.go**

```golang
package abstract_factory

import (
    "errors"
    "fmt"
)

const (
    LuxuryCarType   = 1
    FamiliarCarType = 2
)

type CarFactory struct{}

func (c *CarFactory) GetVehicle(v int) (Vehicle, error) {
    switch v {
    case LuxuryCarType:
        return new(LuxuryCar), nil
    case FamiliarCarType:
        return new(FamiliarCar), nil
    default:
        return nil, errors.New(fmt.Sprintf("Vehicle of type %d not recognized\n", v))
    }
}
```

```golang
package abstract_factory

import (
    "errors"
    "fmt"
)

const (
    SportMotorbikeType  = 1
    CruiseMotorbikeType = 2
)

type MotorbikeFactory struct{}

func (m *MotorbikeFactory) GetVehicle(v int) (Vehicle, error) {
    switch v {
    case SportMotorbikeType:
        return new(SportMotorbike), nil
    case CruiseMotorbikeType:
        return new(CruiseMotorbike), nil
    default:
        return nil, errors.New(fmt.Sprintf("Vehicle of type %d not recognized\n", v))
    }
}
```

- **vehicle.go**

```golang
package abstract_factory

type Vehicle interface {
    GetWheels() int
    GetSeats() int
}
```

- **car.go**

```golang
package abstract_factory

type Car interface {
    GetDoors() int
}
```

- **familiar_car.go**

```golang
package abstract_factory

type FamiliarCar struct{}

func (l *FamiliarCar) GetDoors() int {
    return 5
}

func (l *FamiliarCar) GetWheels() int {
    return 4
}

func (l *FamiliarCar) GetSeats() int {
    return 5
}
```

- **luxury_car.go**

```golang
pacakge abstract_factory

type LuxuryCar struct {}

func (l *LuxuryCar) GetDoors() int {
    return 4
}

func (l *LuxuryCar) GetWheels() int {
    return 4
}

func (l *LuxuryCar) GetSeats() int {
    return 5
}
```

- **motorbike.go**

```golang
package abstract_factory

type Motorbike interface {
    GetType() int
}
```

- **sport_motorbike.go**

```golang
package abstract_factory

type SportMotorbike struct {}

func (s *SportMotorbike) GetWheels() int {
    return 2
}

func (s *SportMotorbike) GetSeats() int {
    return 1
}

func (s *SportMotorbike) GetType() int {
    return SportMotorbikeType
}
```

- **cruise_motorbike.go**

```golang
package abstract_factory

type CruiseMotorbike struct{}

func (c *CruiseMotorbike) GetWheels() int {
    return 2
}

func (c *CruiseMotorbike) GetSeats() int {
    return 2
}

func (c *CruiseMotorbike) GetType() int {
    return CruiseMotorbikeType
}
```

The Abstract factory and Builder patterns can both resolve the same problem, but your particular needs will help you find the slight differences that should lead you to take one solution or the other.

## Prototype design pattern

While with the Builder pattern, we are dealing with repetitive build algorithms and with the factories we are simplifying the creation of many types of objects;
**With the Prototype pattern, we will use an already created instance of some type to clone it and complete it with the particular needs of each context.**

The aim of the Prototype pattern is to have an object or a set of objects that is already created at compilation time, but which you can clone as many times as you want at runtime.

**The key difference between this and a Builder pattern is that objects are cloned for the user instead of building them at runtime.**

### Objective

The main objective for the Prototype design pattern is to avoid repetitive object creation.

- **Maintain a set of objects that will be cloned to create new instances**
- **Provide a default value of some type to start working on top of it**
- **Free CPU of complex object initialization to take more memory resources**

```golang
package creational

import (
    "errors"
    "fmt"
)

type ShirtCloner interface {
    GetClone(m int) (ItemInfoGetter, error)
}

const (
    White = 1
    Black = 2
    Blue  = 3
)

func GetShirtsCloner() ShirtCloner {
    return nil
}

type ShirtsCache struct{}

func (s *ShirtsCache) GetClone(m int) (ItemInfoGetter, error) {
    switch m {
    case White:
        newItem := *whitePrototype
        return &newItem, nil
    case Black:
        newItem := *blackPrototype
        return &newItem, nil
    case Blue:
        newItem := *bluePrototype
        return &newItem, nil
    default:
        return nil, errors.New("Shirt model not recognized")
    }
}

type ItemInfoGetter interface {
    GetInfo() string
}

type ShirtColor byte

type Shirt struct {
    Price float32
    SKU   string
    Color ShirtColor
}

func (s *Shirt) GetInfo() string {
    return fmt.Sprintf("Shirt with SKU '%s' and Color id %d that cost %f\n", s.SKU, s.Color, s.Price)
}

var whitePrototype *Shirt = &Shirt {
    Price: 15.00,
    SKU:   "empty",
    Color: White,
}

var blackPrototype *Shirt = &Shirt {
    Price: 16.00,
    SKU:   "empty",
    Color: Black,
}

var bluePrototype *Shirt = &Shirt {
    Price: 17.00,
    SKU:   "empty",
    Color: Blue,
}

func (i *Shirt) GetPrice() float32 {
    return i.Price
}
```

**The Prototype pattern is a powerful tool to build caches and default objects. You have probably realized too that some patterns can overlap a bit, but they have small differences that make them more appropriate in some cases and not so much in others.**