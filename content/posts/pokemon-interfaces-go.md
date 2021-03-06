+++
title = "GoLang Interfaces Demonstrated with Pokemon"
date = "2021-03-01T06:59:46-06:00"
author = "Jeffrey Serio"
authorTwitter = "hyperreal64" #do not include @
cover = ""
tags = ["golang", "interfaces", "programming"]
keywords = ["golang", "go", "interfaces"]
description = ""
showFullContent = false
toc = true
+++

> I'm not an expert in Go, but I'm writing this for my own learning purposes. It very much a work-in-progress, so please don't take this to be an authoritative source on Go's interfaces. There are bound to be mistakes in this, so I'd be happy to be informed of them. This article assumes the reader has some familiarity with programming and the basics of Go. My intended audience is...well, mostly me! So it's pretty likely that in explaining this to myself, I have taken some things for granted that anyone outside of my own mind would typically want explained.

## What is an interface?
* An abstract type that allows [polymorphism](https://en.wikipedia.org/wiki/Polymorphism_(computer_science)) in Go.
* A protocol for types: a set of rules to which a type must adhere.
* A *skeleton* that consists of the signatures of the methods that should be defined by the type.
* Types that implement an interface *flesh out* the interface's methods.
* For a type to implement an interface, it must define **all** of its methods.

## How to declare and implement an interface
Go differs from other object-oriented languages in that it does not have [classes](https://en.wikipedia.org/wiki/Class_(computer_programming)). Some OOP constructs, such as polymorphism, can be attained implicitly in Go by means of interfaces. The simplest way to think of an interface is as a contract or specification for a [data type](https://en.wikipedia.org/wiki/Data_type). It is an abstract type, in contrast to concrete types like int, string, float, and bool. An interface simply tells the type _what_ action(s) / method(s) the object is allowed to execute. Types that implement an interface define _how_ those action(s) / method(s) are carried out. I will use the classic starter Pokemon to demonstrate how this works.

In Go, there is no *implements* keyword like there is with interfaces in Java. Go's compiler uses the duck test to check for interface compliance: if it *looks*, *walks*, *quacks*, and does all the things that we can reasonably expect a duck to do, then for all intents and purposes it *is* a duck.

In the most abstract, general sense, a Pokemon is a creature that *does things*. One of the things it does is attack other Pokemon in battle. So we can declare a Pokemon attacker interface as follows:

{{< code >}}type pokemon interface {
    attack() string
}{{< /code >}}

Now that we have specified _what_ a Pokemon should do--namely, attack--we need to define a method for _how_ a Pokemon should go about it. First, we need to declare a type that uses the `pokemon` interface. Let's think of this type as the type of Pokemon, e.g., grass, water, fire, etc.

{{< code >}}type grass struct {
    Name string
    GrassAtk string
}{{< /code >}}


Now that we have a `grass` type, let's define the `attack` method:

{{< code >}}func (g grass) attack() string {
    return fmt.Sprintf("%s uses %s!\n", g.Name, g.GrassAtk)
}{{< /code >}}

This method will get the name and attack of the grass type Pokemon and return a string describing the attack action. Now let's give the grass type a concrete value. For that, we'll use Bulbasaur:

{{< code >}}bulbasaur := grass{
    Name: "Bulbasaur",
    GrassAtk: "vine whip",
}{{< /code >}}

To demonstrate what happens when a method does not belong to an interface, we'll define a `pokeCry()` method, which simply prints the Pokemon's name as an exclamation to emulate its cry. Unfortunately we cannot account for inflection and intonation in an easy way, so for the purposes of this demo we'll just use the Pokemon's name. We can combine the above code in a single file:

{{< code >}}// main.go

package main

import "fmt"

type pokemon interface {
	attack() string
}

type grass struct {
	Name     string
	GrassAtk string
}

func (g grass) attack() string {
	return fmt.Sprintf("%s uses %s!\n", g.Name, g.GrassAtk)
}

func (g grass) pokeCry() string {
	return fmt.Sprintf("%s!\n", g.Name)
}

func isAPokemon(p pokemon) {
	fmt.Println(p.attack())
	fmt.Println(p.pokeCry())
}

func main() {
	bulbasaur := grass{
		Name:     "Bulbasaur",
		GrassAtk: "vine whip",
	}

	// implicit: checks if bulbasaur as a pokemon type can use the attack() and pokeCry() methods.
	fmt.Println(isAPokemon(bulbasaur))
}{{< /code >}}

The `isAPokemon` function accepts a pokemon type. The body of the function calls the `attack()` and `pokeCry()` methods. The `attack()` method belongs to the `pokemon` interface, but the `pokeCry()` method does not, so if we try to run this we get an error that `type pokemon has no field or method pokeCry`. Let's fix that by adding the `pokeCry()` method signature to the `pokemon` interface.

{{< code >}}type pokemon interface {
	attack() string
	pokeCry() string
}{{< /code >}}

We see that "Bulbasaur uses vine whip!" and "Bulbasaur!" are both printed to the console.

You can run the above code in the [Go Playground](https://play.golang.org/p/G-Lc2-Iy1GJ).

We now declare other types of Pokemon:

{{< code >}}type water struct {
    Name string
    WaterAtk string
}

type fire struct {
    Name string
    FireAtk string
}{{< /code >}}

And define them with concrete values:

{{< code >}}squirtle := water{
    Name: "Squirtle",
    WaterAtk: "water gun",
}

charmander := fire{
    Name: "Charmander",
    FireAtk: "ember",
}{{< /code >}}

Pokemon battle with other Pokemon, so we have to change the interface's `attack()` signature to accept another Pokemon to attack. Since each Pokemon is defined as concrete values of its type and are not types themselves, and the `attack()` method needs to take one argument to satisfy its signature, we'll use the empty interface, `interface{}`. 

## The empty interface
By definition, the empty interface has zero methods, and since a type has to be able to satisfy all methods of an interface to implement that interface, it follows that all types can implement the empty interface. So using the empty interface as a parameter for the `attack()` method would allow us to pass Pokemon of *any* type to the method.

{{< code >}}type pokemon interface {
    attack(i interface{}) string
}{{< /code >}}

In the world of Pokemon, each type has strengths and weaknesses against other types. Grass Pokemon are strong against water type but weak against fire; water type Pokemon are strong against fire but weak against grass; fire type Pokemon are strong against grass but weak against water. So each Pokemon's attack will have different effects on each opponent. We can implement this by putting each type in its own file, and defining a custom `attack()` method for each one. So now, we have the following *five* separate files. We'll move these files to a directory named 'pokemon'.

{{< code >}}// pokemon/main.go

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
}{{< /code >}}

{{< code >}}// pokemon/pokemon.go

package main

type pokemon interface {
	attack(i interface{}) string
}{{< /code >}}

{{< code >}}// pokemon/grass.go

package main

import "fmt"

type grass struct {
	Name     string
	GrassAtk string
}

func (g grass) attack(i interface{}) string {
	return fmt.Sprintf("%s uses %s against %s!\n", g.Name, g.GrassAtk, i)
}{{< /code >}}

{{< code >}}// pokemon/water.go

package main

type water struct {
	Name     string
	WaterAtk string
}

func (w water) attack(i interface{}) string {
	return fmt.Sprintf("%s uses %s against %s!\n", w.Name, w.WaterAtk, i)
}{{< /code >}}

{{< code >}}// pokemon/fire.go

package main

type fire struct {
	Name    string
	FireAtk string
}

func (f fire) attack(i interface{}) string {
	return fmt.Sprintf("%s uses %s against %s!\n", f.Name, f.FireAtk, i)
}{{< /code >}}

If we run the contents of this directory right now, we get the following result printed to the console:
{{< command-line data-user="ash" data-host="kanto" >}}cd pokemon/
go run .
(out)Bulbasaur uses vine whip against {Squirtle water gun}!
(out)
(out)Squirtle uses water gun against {Charmander ember}!
(out)
(out)Bulbasaur uses vine whip against {Charmander ember}!{{< /command-line >}}

The empty `interface{}` accepts any type as an argument, so we can pass any Pokemon type to its call from the `main()` function, but will only work as long as the type contains fields for name and attack.

If we run this in the console, we see that the entire type value is printed for the Pokemon receiving the attack, which is not the result we want. We want only to display the Pokemon's name, as well as a description of the attack's effectiveness--whether it was 'super-effective' or 'not very effective'--depending on its type. To do this, we need to customize each type's `attack()` method implementation using a Go feature called **type switch**, which is a switch statement for types.

## Type switch on an interface

We can then define `super` and `notVery` as constants, and `effect` and `oppName` as variables in `pokemon.go`. These constants and variables are used in the attack method's definition, and we define them here to avoid repeting them for each type. They are just strings to use in the attack method's return value as parts of a formatted output string.
{{< code >}}// pokemon/pokemon.go

package main

const (
	super = "super-effective!"
	notVery = "not very effective."
)

var effect, oppName string{{< /code >}}

Here is what these constants and variables do:
* `super` is a constant value that describes when the attack is super-effective.
* `notVery` is a constant value that describes when the attack is not-very-effective.
* `effect` is a variable that holds the value of one of the above constants, depending on the Pokemon type's weakness.
* `oppName` is a variable that holds the value of the opponent Pokemon's name.

Now we update each type's attack method to use the type switch, and use type assertion to extract the Name value of the specified type.

{{< code >}}// pokemon/fire.go

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
}{{< /code >}}

{{< code >}}// pokemon/water.go

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
}{{< /code >}}

{{< code >}}// pokemon/grass.go

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
}{{< /code >}}

In the example `main.go` above, `bulbasaur` calls the `attack` method on `squirtle`. Bulbasaur is a grass type, so it will use the attack method defined in `grass.go`. This attack method has an empty interface parameter, which means it can accept *any* type. When we pass `squirtle` to the attack method, the type switch checks its type with `i.(type)`. Squirtle is a water type, so the `case water:` statement gets executed and the other case statements are ignored. Then the `effect` variable takes the value of the `super` constant, and `oppName` takes the value of the Name field defined for `squirtle`, which is "Squirtle". Finally, the attack method returns the string "Bulbasaur uses vine whip on Squirtle. It's super-effective!", which gets printed to the console.

When we run the program, we get the following output.
{{< command-line data-user="ash" data-host="kanto" >}}go run .
(out)Bulbasaur uses vine whip on Squirtle. It's super-effective!
(out)
(out)Squirtle uses water gun on Charmander. It's super-effective!
(out)
(out)Bulbasaur uses vine whip on Charmander. It's not very effective.{{< /command-line >}}


## The Beauty of Abstraction

> The bigger the interface, the weaker the abstraction. --Rob Pike, [Gopherfest SV 2015](https://www.youtube.com/watch?v=PAAkCSZUG1c)

Let's say we have a collection of Pokemon of various types, and we want to list their stats. We re-declare the `pokemon` interface to include a method called `stats()`. We also declare a new type called `list` as a slice of `pokemon` types:

{{< code >}}// pokemon/pokemon.go
package main

import "fmt"

const (
	super   = "super-effective!"
	notVery = "not very effective."
)

var effect, oppName string

type pokemon interface {
	stats()
}

type list []pokemon

func (l list) stats() {
	for _, v := range l {
		v.stats()
		fmt.Println()
	}
}{{< /code >}}

Now in `main.go`, we declare a variable of type `list` named `collection`, and we append to it each of the Pokemon values:
{{< code >}}// pokemon/main.go

package main

func main() {
	bulbasaur := grass{
		Name:     "bulbasaur",
		GrassAtk: "vine whip",
	}

	squirtle := water{
		Name:     "squirtle",
		WaterAtk: "water gun",
	}

	charmander := fire{
		Name:    "charmander",
		FireAtk: "ember",
	}

	var collection list
	collection = append(collection, bulbasaur, squirtle, charmander)

	collection.stats(){{< /code >}}

If we try to run this program, the compiler will inform us that each of the Pokemon in `collection` are missing a corresponding `stats()` method. We see an error like this:
{{< code >}}cannot use bulbasaur (variable of type grass) as pokemon value in argument to append: missing method stats{{< /code >}}

This is because we haven't defined the `stats()` method for each Pokemon type. Let's do that now by adding the following at the bottom of `grass.go`, `water.go`, and `fire.go`:

{{< code >}}// pokemon/grass.go
...

func (g grass) stats() {
	fmt.Printf("Name: %s\nAttack: %s\n", g.Name, g.GrassAtk)
}{{< /code >}}

{{< code >}}// pokemon/water.go
...
func (w water) stats() {
	fmt.Printf("Name: %s\nAttack: %s\n", w.Name, w.WaterAtk)
}{{< /code >}}

{{< code >}}// pokemon/fire.go
...
func (f fire) stats() {
	fmt.Printf("Name: %s\nAttack: %s\n", f.Name, f.FireAtk)
}{{< /code >}}

We run the program and get the following output:
{{< command-line data-user="ash" data-host="kanto" >}}go run .
(out)Name: bulbasaur
(out)Attack: vine whip
(out)
(out)Name: squirtle
(out)Attack: water gun
(out)
(out)Name: charmander
(out)Attack: ember{{< /command-line >}}

By standardizing our `pokemon` interface, we have reduced the amount of code we have to put in `main.go` to print the stats of all our Pokemon. We can define new Pokemon types like psychic, poison, and electric without having to change or add code to `pokemon/pokemon.go`, as long as we define the `stats()` methods for each of them. 

## Recap

* Interfaces are a contract for types. They tell types *what* action(s) / methods a receiver object should execute, while the types are responsible for defining *how* those actions are to be carried out. 
* To implement an interface, a type must define **all** of the interface's methods. Go's compiler uses the duck test to check for interface compliance: if it looks, walks, quacks, and does everything one can reasonably expect a duck to do, then for all intents and purposes it *is* a duck.
* The empty `interface{}` has zero methods, so all types can implement the empty interface.
* Type switch checks the type that implements the given interface. 
