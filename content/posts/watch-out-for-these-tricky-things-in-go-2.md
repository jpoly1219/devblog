---
title: "Watch Out for These Tricky Things in Go 2"
date: 2022-03-06T22:01:35+09:00
draft: false
---

This is a continuation from last week's post, *Watch out For These Tricky Things in Go*. Hope you enjoy!

### An empty interface can be tricky to use.

Many beginners tend to get confused by the concept of the empty interface. I was no exception and struggled with it for a while as well.

A quick primer: an interface in Go depicts a set of methods that do similar things. Any type that implements these methods implement that interface. 

```go
package main

import "fmt"

// any type that has makeSound() is a soundMaker.
type soundMaker interface {
    makeSound()
}

type car struct {
    sound string
}

func (c car) makeSound(){
    fmt.Println(c.sound)
}

type person struct {
    voice string
}

func (p person) makeSound(){
    fmt.Println(p.voice)
}

func main() {
    c := car{"vroom"}
    p := person{"lalala"}

    // car and person type both have makeSound() methods,
    // which means that they implement the soundMaker interface.
    makeSoundThreeTimes(c)
    makeSoundThreeTimes(p)
}

func makeSoundThreeTimes(soundMaker soundMaker) {
    soundMaker.makeSound()
    soundMaker.makeSound()
    soundMaker.makeSound()
}
```

```
vroom
vroom
vroom
lalala
lalala
lalala
```

Therefore, an empty interface is implemented by all types, because there are no methods required to implement it. However, it is really easy to misunderstand this statement, because according to this, it seems like empty interfaces can act like all types, like how generics would.

Take a look at this piece of code:

```go
package main

import "fmt"

func main() {
    intSlice := []int{1, 2, 3}
    printElements(intSlice)
}

func printElements(list []interface{}) {
    for _, v := range list {
        fmt.Println(v)
    }
}
```

You would expect something like this:

```go
1
2
3
```

But you get this instead:

```
cannot use intSlice (type []int) as type []interface {} in argument to printElements
```

Interesting, isn't it? Because `int` satisfies the empty interface, you'd expect `[]int` to be passed as `[]interface{}`. So why would this error out?

You need to see `interface{}` as its own type and not as an alias for or an equivalent of other types. The reason why you cannot just convert from `[]interface{}` to `[]int` is that they are represented differently in memory. You will need to write a separate code for this conversion. Fortunately, it isn't very difficult:

```go
package main

import "fmt"

func main() {
    intSlice := []int{1, 2, 3}
    interfaceSlice:= make([]interface{}, 0)
    for _, v := range intSlice {
        interfaceSlice= append(interfaceSlice, v)
    }
    printElements(interfaceSlice)
}

func printElements(list []interface{}) {
    for _, v := range list {
        fmt.Println(v)
    }
}
```

```
1
2
3
```

Keep in mind, a list of empty interfaces is not an empty interface, therefore will not be able to be implemented by other data types.

### Appending to slices yield `<nil> <nil> data`?

Let's say that you want to append some items to a slice. The code would look like this:

```go
package main

import "fmt"

func main() {
    intSlice := make([]int, 3)
    intSlice = append(intSlice, 1, 2, 3)
    fmt.Println(intSlice)

    interfaceSlice := make([]interface{}, 3)
    interfaceSlice = append(interfaceSlice, "hello", 1, true)
    fmt.Println(interfaceSlice)
}
```

You would expect this to return something like this:

```
[1 2 3]
[hello 1 true]
```

But if we actually run it, this is what we get:

```
[0 0 0 1 2 3]
[<nil> <nil> <nil> hello 1 true]
```

It might seem a bit weird, but the way this work is: 

- Go will allocate space when using the `make()` function. For this example, the size of the int slice will be 3.

- These spaces will be assigned zero-values of the chosen data type (`0` for `int`, `nil` for `interface{}`).

- So because there are already three elements inside the slice, when you try to append to it, the new elements will be appended after the already existing elements.

To prevent this from happening, we can do either of the following.

Firstly, you can specify both length and capacity.

```go
package main

import "fmt"

func main() {
    intSlice := make([]int, 0, 3)
    intSlice = append(intSlice, 1, 2, 3)
    fmt.Println(intSlice)

    interfaceSlice := make([]interface{}, 0, 3)
    interfaceSlice = append(interfaceSlice, "hello", 1, true)
    fmt.Println(interfaceSlice)
}
```

```
[1 2 3]
[hello 1 true]
```

The length determines how many elements are currently in the slice, and the capacity determines how many items it can hold. Think of a slice as a bowl that holds apples, where its length is the number of apples, and the capacity is how many apples it can hold at a time. If more apples were to be added, you would need more bowls.

Secondly, you can use indexes instead.

```go
package main

import "fmt"

func main() {
    intSlice := make([]int, 3)
    for i := range intSlice {
        intSlice[i] = i + 1
    }
    fmt.Println(intSlice)

    interfaceSlice := make([]interface{}, 3)
    interfaceToAdd := []interface{}{"hello", 1, true}
    for i := range interfaceSlice {
        interfaceSlice[i] = interfaceToAdd[i]
    }
    fmt.Println(interfaceSlice)
}
```

```
[1 2 3]
[hello 1 true]
```

This is more straightforward because you index the given slice, and then assign elements to that location.

### Conclusion

I hope this post helped you with your Go programming journey. If you are getting started with Go, there are plenty of other resources that you may wish to look into as well. Here are some I used to learn Go.

- [Go by Example](https://gobyexample.com)

- [A Tour of Go](https://go.dev/tour/list)

- [Calhoun.io](https://www.calhoun.io)

The topics in this post are very interesting because they deal with how Go works under the hood. I am planning on writing a more in-depth guide on these data types and some quirks of Go, so definitely keep an eye out for those.

Thank you for reading! You can read this on [Dev.to]([Watch out For These Tricky Things in Go - Part 2 - DEV Community](https://dev.to/jpoly1219/watch-out-for-these-tricky-things-in-go-part-2-3k8e)) and [Medium](https://medium.com/@jpoly1219/watch-out-for-these-tricky-things-in-go-part-2-6a533703a44d).
