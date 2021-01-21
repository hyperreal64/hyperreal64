+++
title = "GoLang Interfaces Demonstrated with Pokemon"
date = "2021-01-20T18:52:26-06:00"
author = "Jeffrey Serio"
authorTwitter = "hyperreal64" #do not include @
cover = ""
tags = ["golang", "interfaces", "programming"]
keywords = ["golang", "go", "interfaces"]
description = ""
showFullContent = false
+++

> This article assumes the reader has some familiarity with programming and the basics of Go.

> I'm not an expert in Go, but I'm writing this for my own learning purposes. Please don't take this to be an authoritative source on Go's interfaces. :-)

## What is an interface?
* An abstract concept that allows [polymorphism](https://en.wikipedia.org/wiki/Polymorphism_(computer_science)) in Go.
* A protocol for types: a set of rules to which a type must adhere.
* A *skeleton* that consists of the signatures of the methods that should be defined by the type.
* Types that implement an interface *flesh out* the interface's methods.
* For a type to implement an interface, it must define **all** of its methods.

## How to declare and implement an interface
Go differs from other object-oriented languages in that it does not have [classes](https://en.wikipedia.org/wiki/Class_(computer_programming)). Some OOP constructs, such as polymorphism, can be attained implicitly in Go by means of interfaces. The simplest way to think of an interface is as a contract or specification for a [data type](https://en.wikipedia.org/wiki/Data_type), while the data type describes some aspect of a real-world object that _does things_. An interface simply tells the type _what_ the object does. Types that implement an interface define _how_ an object should do the things. I will use the classic starter Pokemon to demonstrate how this works.

In the most abstract, general sense, a Pokemon is a creature that *does things*. One of the things it does is attack other Pokemon in battle. So we can declare a Pokemon interface as follows:

```go
type pokemon interface {
    attack()
}
```

Now that we have specified _what_ a Pokemon should do--namely, attack--we need to define a method for _how_ a Pokemon should go about it. First, we need to declare a type that uses the `pokemon` interface. Let's think of this type as the type of Pokemon, e.g., grass, water, fire, etc.

```go
type grass struct {
    Name string
    GrassAtk string
}
```

Now that we have a `grass` type, let's define the `attack` method:

```go
func (g grass) attack() string {
    return fmt.Sprintf("%s uses %s!\n", g.Name, g.GrassAtk)
}
```

This method will get the name and attack of the grass type Pokemon and return a string describing the attack action. Now let's give the grass type a concrete value. For that, we'll use Bulbasaur:

```go
bulbasaur := grass{
    Name: "Bulbasaur",
    GrassAtk: "vine whip",
}
```

We can combine the above code in a single file:

```go
// main.go

package main

import "fmt"

type pokemon interface {
	attack()
}

type grass struct {
	Name     string
	GrassAtk string
}

func (g grass) attack() string {
	return fmt.Sprintf("%s uses %s!\n", g.Name, g.GrassAtk)
}

func main() {
	bulbasaur := grass{
		Name:     "Bulbasaur",
		GrassAtk: "vine whip",
	}

	fmt.Println(bulbasaur.attack())
}
```

If we run this program in the console, we get the following result:
{{< command-line data-user="ash" data-host="kanto" >}}go run main.go
(out)Bulbasaur uses vine whip!{{< /command-line >}}

You can run the above code in the [Go Playground](https://play.golang.org/p/UkEeEOuuqpw).

We now declare other types of Pokemon:

```go
type water struct {
    Name string
    WaterAtk string
}

type fire struct {
    Name string
    FireAtk string
}
```

And define them with concrete values:

```go
squirtle := water{
    Name: "Squirtle",
    WaterAtk: "water gun",
}

charmander := fire{
    Name: "Charmander",
    FireAtk: "ember",
}
```

Pokemon battle with other Pokemon, so we have to change the interface's `attack()` signature to accept another Pokemon to attack. Since each Pokemon is defined as concrete values of its type and are not types themselves, and the `attack()` method needs to take one argument to satisfy its signature, we'll use the empty interface, `interface{}`. 

## The empty interface
By definition, the empty interface has zero methods, and since a type has to be able to satisfy all methods of an interface to implement that interface, it follows that all types can implement the empty interface. So using the empty interface as a parameter for the `attack()` method would allow us to pass Pokemon of *any* type to the method.

```go
type pokemon interface {
    attack(i interface{})
}
```

In the world of Pokemon, each type has strengths and weaknesses against other types. Grass Pokemon are strong against water type but weak against fire; water type Pokemon are strong against fire but weak against grass; fire type Pokemon are strong against grass but weak against water. So each Pokemon's attack will have different effects on each opponent. We can implement this by putting each type in its own file, and defining a custom `attack()` method for each one. So now, we have the following *five* separate files. We'll move these files to a directory named 'pokemon'.

```go
// pokemon/main.go

package main

import "fmt"

func main() {
	bulbasaur := grass{
		Name:     "Bulbasaur",
		GrassAtk: "vine whip",
	}

	squirtle := water{
		Name:     "Squirtle",
		WaterAtk: "water gun",
	}

	charmander := fire{
		Name:    "Charmander",
		FireAtk: "ember",
	}

	fmt.Println(bulbasaur.attack(squirtle))
	fmt.Println(squirtle.attack(charmander))
	fmt.Println(bulbasaur.attack(charmander))
}
```

```go
// pokemon/pokemon.go

package main

type pokemon interface {
	attack(i interface{})
}
```

```go
// pokemon/grass.go

package main

import "fmt"

type grass struct {
	Name     string
	GrassAtk string
}

func (g grass) attack(i interface{}) string {
	return fmt.Sprintf("%s uses %s against %s!\n", g.Name, g.GrassAtk, i)
}
```

```go
// pokemon/water.go

package main

type water struct {
	Name     string
	WaterAtk string
}

func (w water) attack(i interface{}) string {
	return fmt.Sprintf("%s uses %s against %s!\n", w.Name, w.WaterAtk, i)
}
```

```go
// pokemon/fire.go

package main

type fire struct {
	Name    string
	FireAtk string
}

func (f fire) attack(i interface{}) string {
	return fmt.Sprintf("%s uses %s against %s!\n", f.Name, f.FireAtk, i)
}
```

If we run the contents of this directory right now, we get the following result printed to the console:
{{< command-line data-user="ash" data-host="kanto" >}}cd pokemon/
go run .
(out)Bulbasaur uses vine whip against {Squirtle water gun}!
(out)
(out)Squirtle uses water gun against {Charmander ember}!
(out)
(out)Bulbasaur uses vine whip against {Charmander ember}!{{< /command-line >}}

We can see that the entire type value is printed for the Pokemon receiving the attack, which is not the result we want. We want only to display the Pokemon's name, as well as a description of the attack's effectiveness--whether it was 'super-effective' or 'not very effective'--depending on its type. To do this, we need to customize each type's `attack()` method implementation using a Go feature called **type switch**, which is a switch statement for types.

## Type switch on an interface

We can then define `super` and `notVery` as constants, and `effect` and `oppName` as variables in `pokemon.go`. These constants and variables are used in the attack method's definition, and we define them here to avoid repeting them for each type. They are just strings to use in the attack method's return value as parts of a formatted string.
```go
// pokemon/pokemon.go

package main

const (
	super = "super-effective!"
	notVery = "not very effective."
)

var effect, oppName string
```

Here is what these constants and variables do:
* `super` is a constant value that describes when the attack is super-effective.
* `notVery` is a constant value that describes when the attack is not-very-effective.
* `effect` is a variable that holds the value of one of the above constants, depending on the Pokemon type's weakness.
* `oppName` is a variable that holds the value of the opponent Pokemon's name.

Now we update each type's attack method to use the type switch, and use type assertion to extract the Name value of the specified type.

```go
// pokemon/fire.go

func (f fire) attack(i interface{}) string {
	switch i.(type) {
	case grass:
		effect = super
		oppName = i.(grass).Name
	case water:
		effect = notVery
		oppName = i.(water).Name
	case fire:
		effect = notVery
		oppName = i.(fire).Name
	}

	return fmt.Sprintf("%s uses %s on %s. It's %s\n", f.Name, f.FireAtk, oppName, effect)
}
```

```go
// pokemon/water.go

func (w water) attack(i interface{}) string {
	switch i.(type) {
	case fire:
		effect = super
		oppName = i.(fire).Name
	case grass:
		effect = notVery
		oppName = i.(grass).Name
	case water:
		effect = notVery
		oppName = i.(water).Name
	}

	return fmt.Sprintf("%s uses %s on %s. It's %s\n", w.Name, w.WaterAtk, oppName, effect)
}
```

```go
// pokemon/grass.go

func (g grass) attack(i interface{}) string {

	switch i.(type) {
	case water:
		effect = super
		oppName = i.(water).Name
	case fire:
		effect = notVery
		oppName = i.(fire).Name
	case grass:
		effect = notVery
		oppName = i.(grass).Name
	}

	return fmt.Sprintf("%s uses %s on %s. It's %s\n", g.Name, g.GrassAtk, oppName, effect)
}
```

In the example `main.go` above, `bulbasaur` calls the `attack` method on `squirtle`. Bulbasaur is a grass type, so it will use the attack method defined in `grass.go`. This attack method has an empty interface parameter, which means it can accept *any* type. When we pass `squirtle` to the attack method, the type switch checks its type with `i.(type)`. Squirtle is a water type, so the `case water:` statement gets executed and the other case statements are ignored. Then the `effect` variable takes the value of the `super` constant, and `oppName` takes the value of the Name field defined for `squirtle`, which is "Squirtle". Finally, the attack method returns the string "Bulbasaur uses vine whip on Squirtle. It's super-effective!", which gets printed to the console.

When we run the program, we get the following output.
{{< command-line data-user="ash" data-host="kanto" >}}go run .
(out)Bulbasaur uses vine whip on Squirtle. It's super-effective!
(out)
(out)Squirtle uses water gun on Charmander. It's super-effective!
(out)
(out)Bulbasaur uses vine whip on Charmander. It's not very effective.{{< /command-line >}}

## Recap

* Interfaces are a contract for types. They tell types *what* the receiver object should do, while the types are responsible for defining *how* to do it. To implement an interface, a type must define **all** of the interface's methods.
* The empty `interface{}` has zero methods, and thus all types can implement the empty interface.
* Type switch checks the type that implements the given interface. This is useful for when want to operate on data based on its type. 