---
title: "Intro to Concurrency in Go"
date: 2022-08-14T16:00:39+09:00
draft: false
---

Concurrency is a cool topic that can be a huge asset once you get the hang of it. To be honest, I was scared to write this post at first because I myself wasn't too comfortable with concurrency until recently. I got the basics down, so I wanted to help other beginners learn concurrency in Go. This is the first of many concurrency tutorials, so stay tuned for more!

### What is concurrency and why does it matter?

Concurrency is the ability to run multiple things at the same time. Your computer has a CPU. A CPU has several threads. Each thread usually runs one program at a time. When we normally write code, that code runs sequentially, meaning that each job is run back to back. In concurrent code, those jobs are run simultaneously by the thread.

A good analogy is the one to a home cook. I still remember the first time I tried to cook pasta. I followed the recipe step-by-step. I chopped the vegetables, made the sauce, then cooked the spaghetti, then mixed the two. Here, every step was done sequentially, so the next job had to wait until the current job was done.

Fast forward to now, where I became more experienced at cooking pasta. I now start the spaghetti first, and then work on the sauce in the meantime. Cooking time was reduced to almost half because cooking the spaghetti and the sauce happened concurrently.

### Concurrency vs. Parallelism

Concurrency is a bit different from parallelism. Parallelism is similar to concurrency in that multiple jobs are happening at once. However, in parallelism, multiple threads are working different jobs each, whereas, in concurrency, one thread is juggling between different jobs.

So concurrency and parallelism are two different concepts. A program can run both concurrently and parallel. Your code can be written either sequentially or concurrently. That code can be run on a single-core machine or a multi-core machine. Think of concurrency as a characteristic of your code, while parallelism as a characteristic of the execution.

### Goroutines, the worker Mortys

Go makes it very simple to write concurrent code. Each concurrent job is represented by a *goroutine*. You can start a goroutine by using the `go` keyword before a function call. Ever watched *Rick and Morty*? Imagine your `main` function as a Rick who delegates tasks to goroutine Mortys.

Let's start with a sequential code.

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    simple()
}

func simple() {
    fmt.Println(time.Now(), "0")
    time.Sleep(time.Second)

    fmt.Println(time.Now(), "1")
    time.Sleep(time.Second)

    fmt.Println(time.Now(), "2")
    time.Sleep(time.Second)

    fmt.Println("done")
}
```

```
2022-08-14 16:22:46.782569233 +0900 KST m=+0.000033220 0
2022-08-14 16:22:47.782728963 +0900 KST m=+1.000193014 1
2022-08-14 16:22:48.782996361 +0900 KST m=+2.000460404 2
done
```

The code above prints out the current time along with a string. Each print statement takes one second to run. In total, the code took around three seconds to complete.

Now let's compare that to a concurrent code.

```go
func main() {
    simpleConc()
}

func simpleConc() {
    for i := 0; i < 3; i++ {
        go func(index int) {
            fmt.Println(time.Now(), index)
        }(i)
    }

    time.Sleep(time.Second)
    fmt.Println("done")
}
```

```
2022-08-14 16:25:14.379416226 +0900 KST m=+0.000049175 2
2022-08-14 16:25:14.379446063 +0900 KST m=+0.000079012 0
2022-08-14 16:25:14.379450313 +0900 KST m=+0.000083272 1
done
```

The code above fires off three goroutines that each print the current time and `i`. The program waits for a second, and exits. This code took around one second to complete. That's about three times faster than the sequential version.

"Hold on," I hear you ask. "Why wait a whole second? Couldn't we remove that line to make the program run as fast as possible?" Good question! Let's see what happens.

```go
func main() {
    simpleConcFail()
}

func simpleConcFail() {
    for i := 0; i < 3; i++ {
        go func(index int) {
            fmt.Println(time.Now(), index)
        }(i)
    }

    fmt.Println("done")
}
```

```
done
```

Hmm... The program did exit without any panics, but we are missing the output from the goroutines. Why were they skipped?

This is because by default, Go does not wait for goroutines to finish. Did you know that `main` is also run inside a goroutine? The `main` goroutine fires off the worker goroutines by calling `simpleConcFail`, but it exits before the workers can even finish their job.

Let's go back to the cooking analogy. Imagine you have three chefs each responsible for cooking the sauce, the spaghetti, and the meatball. Now, imagine if **Go**rdon Ramsey orders the chefs to cook a plate of spaghetti & meatballs. The three chefs will work hard to cook the sauce, the spaghetti, and the meatballs. But before the chefs are even done, Gordon rings the bell and orders the waiter to serve the food. Obviously, the food isn't ready, and the customers will only get an empty plate.

This is why we wait for a second before exiting the program. We aren't always sure that every job will finish in one second. There is a better way to wait for jobs to be done, but we first need to learn another concept.

To summarize, we learned these things:

- Jobs are delegated to goroutines.

- Using concurrency can boost your performance.

- The `main` goroutine does not wait for worker goroutines to finish by default.

- We need a way to wait for each goroutine to finish.

### Channels, the green portal

How do goroutines communicate with each other? Through channels, of course. Channels act like portals. You can send and receive data through channels. Here's how you make a channel in Go.

```go
ch := make(chan int)
```

Every channel is strongly typed, and will only allow data of that type to pass through. Let's see how we can use this.

```go
func main() {
    unbufferedCh()
}

func unbufferedCh() {
    ch := make(chan int)

    go func() {
        ch <- 1
    }()

    res := <-ch
    fmt.Println(res)
}
```

```
1
```

Simple, right? We make a channel named `ch`. We have a goroutine that sends `1` to `ch`, and we receive that data and save it to `res`.

Why do we need a goroutine here, you ask? Because not doing so will result in a deadlock.

```go
func main() {
    unbufferedChFail()
}

func unbufferedChFail() {
    ch := make(chan int)
    ch <- 1
    res := <-ch
    fmt.Println(res)
}
```

```
fatal error: all goroutines are asleep - deadlock!
```

We run into a new word. What is a deadlock? A deadlock is where your program is stuck. Why would the above code get stuck in a deadlock?

To understand this, we need to know an important characteristic of channels. We created an unbuffered channel, meaning that nothing can be stored within it at a given time. This means that both the sender and the receiver must be ready simultaneously before data can be transferred across the channel.

In the failed example, the send and receive actions happen sequentially. We send `1` to `ch`, but at that time there is nobody to receive the data. The receiving happens at a later line, meaning that `1` cannot be sent until the receiving line is run. Sadly, `1` can't be sent first because `ch` is unbuffered and has no space to hold any data.

In the working example, the send and receive actions happen simultaneously. The `main` function fires off the goroutine and tries to receive from `ch`. At that time, the goroutine is sending `1` to `ch`. Therefore this code can run without deadlock.

One other way to receive from a channel without deadlocks is to close the channel first.

```go
func main() {
    unbufferedCh()
}

func unbufferedCh() {
    ch2 := make(chan int)
    close(ch2)
    res2 := <-ch2
    fmt.Println(res2)
}
```

```
0
```

Closing the channel means that no more data can be sent to it. We can still receive it from the channel. For unbuffered channels, receiving from a closed channel will return a zero value of the channel's type.

To summarize, we learned these things:

- Channels are the way goroutines communicate with each other.

- You can send and receive data through channels.

- Channels are strongly typed.

- Unbuffered channels have no space to store data, so sending and receiving must happen simultaneously. Otherwise, your code will be stuck in a deadlock.

- A closed channel will not accept any data.

- Receiving from a closed unbuffered channel will return a zero value.

Wouldn't it be nice for channels to hold data for some time? Here's where buffered channels come to play.

### Buffered channels, the portal that is somehow cylindrical?

Buffered channels are channels with buffers. Data can be stored in these, so sending and receiving don't have to happen simultaneously.

```go
func main() {
    bufferedCh()
}

func bufferedCh() {
    ch := make(chan int, 1)
    ch <- 1
    res := <-ch
    fmt.Println(res)
}
```

```
1
```

Here, `1` is stored inside `ch` until we receive it.

Obviously, we can't send more to a full buffered channel. You need to have space in the buffer before you can send more.

```go
func main() {
    bufferedChFail()
}

func bufferedChFail() {
    ch := make(chan int, 1)
    ch <- 1
    ch <- 2
    res := <-ch
    fmt.Println(res)
}
```

```
fatal error: all goroutines are asleep - deadlock!
```

You also cannot receive from an empty buffered channel.

```go
func main() {
    bufferedChFail2()
}

func bufferedChFail2() {
    ch := make(chan int, 1)
    ch <- 1
    res := <-ch
    res2 := <-ch
    fmt.Println(res, res2)
}
```

```
fatal error: all goroutines are asleep - deadlock!
```

If a channel is full, the send operation will wait until there is space available. This is demonstrated here in this code.

```go
func main() {
    bufferedCh2()
}

func bufferedCh2() {
    ch := make(chan int, 1)
    ch <- 1
    go func() {
        ch <- 2
    }()
    res := <-ch
    fmt.Println(res)
}
```

```
1
```

We receive once to take out the `1` so that the goroutine can send `2` to the channel. We didn't receive from `ch` twice, so only `1` will be received.

We can also receive from closed buffered channels. In this case, we can range over the closed channel to iterate over the remaining items inside it.

```go
func main() {
    bufferedChRange()
}

func bufferedChRange() {
    ch := make(chan int, 3)
    ch <- 1
    ch <- 2
    ch <- 3
    close(ch)
    for res := range ch {
        fmt.Println(res)
    }
    // you could also do this
    // fmt.Println(<-ch)
    // fmt.Println(<-ch)
    // fmt.Println(<-ch)
}
```

```
1
2
3
```

Ranging over an open channel will never stop. This means that at some point, the channel will be empty and the range loop will try to receive from an empty channel, resulting in a deadlock.

To summarize,

- Buffered channels are channels with space to hold items.

- Sending and receiving does not have to happen at the same time, unlike unbuffered channels.

- Sending to a full channel and receiving from an empty channel will result in a deadlock.

- You can iterate over a closed channel to receive the remaining values inside the buffer.

### Waiting for Godot... I mean, goroutines to finish, using channels

Channels can be used to sync goroutines. Remember how I told you that the sender and receiver must both be ready before you can transfer data through an unbuffered channel? This means that the receiver will wait until the sender is ready. We can say that receiving is *blocking*, meaning that the receiver will block the rest of the code from running until it has received something. Let's use this nifty trick to sync our goroutines.

```go
func main() {
    basicSyncing()
}

func basicSyncing() {
    done := make(chan struct{})

    go func() {
        for i := 0; i < 5; i++ {
            fmt.Printf("%s worker %d start\n", fmt.Sprint(time.Now()), i)
            time.Sleep(time.Duration(rand.Intn(5)) * time.Second)
        }
        close(done)
    }()

    <-done
    fmt.Println("exiting...")
}
```

We make a `done` channel that is responsible for blocking the code until the goroutine is done. `done` can be of any type, but `struct{}` is used often for these types of channels. Its purpose is not to transfer structs, so its type doesn't matter.

The worker goroutine will close `done` once its job is done. At this point, we can receive from `done`, which will be an empty struct. The receiving action unblocks the code, allowing it to exit.

This is how we wait for goroutines to finish using channels.

### Conclusion

Concurrency can seem like a daunting topic. I certainly believed that to be the case. However, after understanding the basics, I think the implementation is really beautiful. Hopefully, you guys can get something out of this tutorial! We've merely scratched the surface, and there is much more that Go has to offer us. I'll see you next time with more concurrency tutorials. Bye!

You can also read this post on [Medium](https://medium.com/@jpoly1219/intro-to-concurrency-in-go-5cf971a9d0b5) and [Dev.to](https://dev.to/jpoly1219/intro-to-concurrency-in-go-46e4).
