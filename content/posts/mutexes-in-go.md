---
title: "Mutexes in Go"
date: 2022-09-18T20:01:36-04:00
draft: false
---

Welcome back to Intro to Concurrency in Go! Today we will be looking at how to protect shared resources from being overwritten by other goroutines. If you have played Minecraft before, chances are that you couldn't build something because your friends took that one item in the chest before you could use it. Wouldn't it be nice to figure out a way to make sure that only one person can interact with the chest at a given time? Well, it turns out that gamers aren't the only ones who thought of this.

### Mutexes

In Go, cases like these are called "data races." This happens when more than one goroutine tries to access the same data at the same time. As developers, we want to prevent data races as much as possible. If you go around the Go developer community, you may have heard the term "concurrency-safe" be thrown around. This basically means that there are no data races.

To prevent a data race, we can use something called a "mutex." A mutex acts like a switch that we can toggle on or off. When it's on, we are not allowed to touch the variable at hand. When it's off, we can do whatever we want to that variable.

A mutex usually lives inside a struct alongside the variable we would like to modify. So the variable becomes a bit clunkier but receives protection from other goroutines.

### Example code

A code tends to speak a thousand words. Let's see how we can use one.

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    type chest struct {
        mu        sync.Mutex
        container []string
    }

    newChest := chest{container: []string{}}
    newChest.container = append(newChest.container, "Oak Plank", "Spruce Plank", "Birch Plank", "Jungle Plank")
    var wg sync.WaitGroup
    wg.Add(2)

    go func() {
        defer wg.Done()
        newChest.mu.Lock()
        newChest.container = newChest.container[0:2]
        newChest.mu.Unlock()
    }()

    go func() {
        defer wg.Done()
        newChest.mu.Lock()
        if len(newChest.container) == 4 {
            fmt.Println("we can make a crafting table!")
            time.Sleep(time.Second)
            fmt.Println("currently have: ", newChest.container)
            fmt.Println("making a crafting table...")
            fmt.Println("done!")
        }

        newChest.mu.Unlock()
    }()

    wg.Wait()

    fmt.Println(newChest.container)
}
```

```
we can make a crafting table!
currently have:  [Oak Plank Spruce Plank Birch Plank Jungle Plank]
making a crafting table...
done!
[Oak Plank Spruce Plank]
```

Here, we have two goroutines trying to access the chest. The first goroutine wants to take the plank, and the second wants to create a crafting table, which requires four planks. For this run, the second goroutine fired first, which made the first goroutine wait until the second goroutine access the chest first.

What happens if we take out the mutex?

```go
func main() {
    type chest struct {
        mu        sync.Mutex
        container []string
    }

    newChest := chest{container: []string{}}
    newChest.container = append(newChest.container, "Oak Plank", "Spruce Plank", "Birch Plank", "Jungle Plank")
    var wg sync.WaitGroup
    wg.Add(2)

    go func() {
        defer wg.Done()
        // newChest.mu.Lock()
        newChest.container = newChest.container[0:2]
        // newChest.mu.Unlock()
    }()

    go func() {
        defer wg.Done()
        // newChest.mu.Lock()
        if len(newChest.container) == 4 {
            fmt.Println("we can make a crafting table!")
            time.Sleep(time.Second)
            fmt.Println("currently have: ", newChest.container)
            fmt.Println("making a crafting table...")
            fmt.Println("done!")
        }

        // newChest.mu.Unlock()
    }()

    wg.Wait()

    fmt.Println(newChest.container)
}
```

```
we can make a crafting table!
currently have:  [Oak Plank Spruce Plank]
making a crafting table...
done!
[Oak Plank Spruce Plank]
```

What happens here doesn't make sense. We let the first goroutine take the planks before we can even make a crafting table, but our code still said that we can make a crafting table. This is an unexpected behavior caused by data race.

Now you see why mutexes are important in designing concurrent programs. You don't know when what goroutine will modify a certain value, and you do not want any discrepancies happening due to data races. Imagine your user logging out before you can deliver a resource to him. You certainly want the resources delivery goroutine to lock the user authentication status until it's done, right?

### There are different types of mutexes

`sync.Mutex` isn't the only mutex there is. Another frequently used mutex is called the `sync.RWMutex`. RWMutex, which is short for a read-write mutex, is a special type of mutex that allows many goroutines to read from a shared resource at the same time, but only allows one to write to it. This comes in handy when you are going to do a lot of read operations on your resources. A normal mutex will only have the ability to lock out every other goroutine aside from the one currently accessing it, making reading operations very slow. RWMutexes have a special locking method called `RLock` that allows simultaneous read operations. Use whatever floats your boat - I have seen both used widely. Usage is very similar to normal mutexes, so I didn't bother adding a code snippet just for `RLock`. Use it for functions that read from a resource, and make sure to `RUnlock` after you are done. For functions that write to a resource, use `Lock` and `Unlock`.

### Conclusion

Thanks for reading! With this post, the Intro to Concurrency in Go series is finally over! We covered most topics on concurrency in Go. I will come back with a new topic next week, so stay tuned!

This post can be viewed on Medium and Dev.to.
