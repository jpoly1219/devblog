---
title: "Interfaces in Go"
date: 2022-07-17T15:58:15+09:00
draft: false
---

If you are going to use Go extensively, you need to understand how to use interfaces. Interfaces aren't specifically a Go thing, but Go is one of the more extensive users of the feature. Interfaces allow you to write reusable code.

### What are interfaces?

Interfaces are a way to group objects into their common behaviors. An interface is defined by its name and the methods the objects need to define. Any object that has those methods defined "implements" the interface.

For example, let's say that three students A, B, and C can all cook. They are trying to serve me good food. However, since A, B, and C are all trained under different chefs, they cook different things. Nonetheless, they are all chefs who can cook. We can say that A, B, and C implement the "chef" interface because they can all cook in different ways.

Let's see how this can be implemented in Go.

### How does one use it in Go?

Here's the code for the above example.

```go
package main

import (
	"fmt"
	"math/rand"
)

type cook interface {
	cookFood()
	getName() string
}

type chef struct {
	name    string
	cuisine string
}

func (c chef) cookFood() {
	result := fmt.Sprintf("%s the professional chef is cooking %s food", c.name, c.cuisine)
	fmt.Println(result)
}

func (c chef) getName() string {
	return c.name
}

type homeCook struct {
	name string
}

func (h homeCook) cookFood() {
	result := fmt.Sprintf("%s the home cook is cooking food", h.name)
	fmt.Println(result)
}

func (h homeCook) getName() string {
	return h.name
}

func serve(c cook) {
	c.cookFood()
	result := fmt.Sprintf("%s: dinner is served!", c.getName())
	fmt.Println(result)
}

func main() {
	chef1 := chef{"Brian", "Korean"}
	chef2 := chef{"Vincenzo", "Italian"}
	homeCook1 := homeCook{"Amara"}
	homeCook2 := homeCook{"Dana"}

	cooks := []cook{chef1, chef2, homeCook1, homeCook2}

	numCustomers := 100
	for i := 0; i < numCustomers; i++ {
		serve(cooks[rand.Intn(len(cooks))])
	}
}

```

We defined an interface named `cook`. We want anyone who can `cookFood` and `getName` to be able to identify themselves as a `cook`.

Below, we define two structs `chef` and `homeCook`. Both have the `name` field, but only the `chef` has a `cuisine` field to define what cuisine he or she specializes in.

For both `chef` and `homeCook` structs to implement the `cook` interface, they need to both define `cookFood` and `getName` methods. The details of the implementations are straightforward - just a simple print statement.

Now let's look at the `main` function. We define two `chef` objects `chef1` and `chef2`. We also define two `homeCook` objects `homeCook1` and `homeCook2`. Those four objects are stored in our `cooks` slice with the type `cook`, our interface. Normally you wouldn't be able to store objects of different types in a single slice, but using an interface allows us to do this.

We call the `serve` function inside the for loop, which accepts a `cook`. This normally doesn't work - we would need to define two `serve` functions, one for `chef` type and one for `homeCook` type. However, using an interface helps us avoid this repetition.

It can be hard to appreciate interfaces when your codebase is small. Small projects tend not to need a lot of structure, so you may be able to get by without using one. However, interfaces allow for really clean, predictable code as your project grows.

### How does it work under the hood?

Not only are interfaces incredibly useful in designing your code, but they are also really cool in their implementation. Interfaces can be described as two conjoined blocks of pointers: one points at the type definition, and the other points at its underlying value. Confusing, right? Take a look at this line from the example above.

```go
chef1 := chef{"Brian", "Korean"}
```

`chef1` implements the `cook` interface. If we dissect `chef1` in interface form, the first pointer will point to the type definition of the `chef` struct, and the second pointer will point to the actual value of `chef1`.

### What are empty interfaces?

Now we can think about a more advanced, or rather, a more simple concept: the empty interface. We stated above that an interface groups objects by certain behaviors. What would an empty interface look like? It would have no methods for any type to implement. This means that an object of any type can implement the interface. It's like grouping living humans into a category called "organism". Age, height, race, and sex doesn't matter - humans are all organisms.

Take a look at this snippet:

```go
func main() {
    a := "hello"
    b := 100
    c := 3.14
    
    objects := []interface{}{a, b, c}
}
```

This code will compile because `objects` is a slice that stores any object that implements the empty interface.

### Tips

To finish the post, I wanted to share with you some of the tips I have accrued over my short Go development journey.

- Keep in mind that while empty interfaces provide lots of flexibility, you have the responsibility to care for different possible types. For example, let's say that you are unmarshalling a JSON object. Sometimes, you don't know the exact structure of the incoming JSON. Go will cleverly use the type `map[string]interface{}` to unmarshal the JSON object. JSON keys will be stored as `string`, and the values will be stored as `interface{}`. When manipulating the value, you must account for every single possible type, or else your code will flop spectacularly. This is the tax you have to pay when working with empty interfaces.

- Name your interfaces consistently. It helps me a lot when I name my interfaces as `--er`. For example, `copier`, `reader`, `parser`, etc. There is a notable exception to this in the standard library, which is the `builtin.Error` interface. I guess this is ok because it still rhymes with other interfaces that end with `--er`.

- Keeping interface and struct/struct methods definition on a separate file helps readability. This isn't everyone's cup of tea, but I found this to help in many situations.

- You don't have to re-invent the wheel. If you need an interface, see if the standard library already provides it. Some of the most common ones that you can implement are `fmt.Stringer`, `io.Reader`, `io.Writer`, `builtin.Error`, `http.ResponseWriter`, `sort.Interface`, etc.

- An interface is a powerful hammer, but not every problem is a nail. You don't need an interface for everything. Sometimes the overhead and the effort of creating one isn't worth it. Try building your app without one, and you will run into a situation where you want to have an interface.

### Conclusion

I hope this post helped you clear up some of your uncertainties about interfaces in Go. Interfaces are an essential part of the Go programming language, and you will undoubtedly run into them in your journey. When the time comes, I believe that you can handle it well. Keep up the good work gophers, and I'll see you next week with a new post.

This post is also available on Medium and Dev.to.


