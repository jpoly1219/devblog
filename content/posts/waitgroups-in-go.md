---
title: "Waitgroups in Go"
date: 2022-08-21T15:59:17+09:00
draft: true
---

Welcome back to *Introduction to Concurrency in Go*! Last time, we had our first taste of concurrency in Go. We examined what goroutines and channels are, the differences between buffered and unbuffered channels, and a simple way to wait for goroutines to finish. This time, we are going to talk about WaitGroups, which are a different way to sync your goroutines.

### What are WaitGroups?

WaitGroups are an effective way to sync your goroutines. Remember how we used channels to wait for other goroutines to finish? WaitGroups can help you achieve the same goal. Imagine a car trip with your family. Your dad stops by a strip mall or a fast food restaurant to get some food and go to the bathroom. You ideally want to wait for everyone to come back before driving off to the horizon again. WaitGroups help you do this.

WaitGroups are defined by calling the `sync` package in the Go standard library.

```go
var wg sync.WaitGroup
```

So what is a WaitGroup even? A WaitGroup is basically a struct that holds certain information about how many goroutines the program needs to wait on. It's a *group* that contains the number of goroutines that you need to *wait* on.

WaitGroups have three most important methods: `Add`, `Done`, and `Wait`.

- `Add` adds to the total amount of goroutines you need to wait on.

- `Done` subtracts one from the total amount of goroutines you need to wait on.

- `Wait` blocks the code from continuing until there are no more goroutines to wait on.

### How to use WaitGroups

Let's see an example code snippet.

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var wg sync.WaitGroup
	wg.Add(1)

	go func() {
		defer wg.Done()

		fmt.Println(time.Now(), "start")
		time.Sleep(time.Second)
		fmt.Println(time.Now(), "done")
	}()
	
	wg.Wait()
	fmt.Println(time.Now(), "exiting...")
}
```

```
2022-08-21 17:01:54.184744229 +0900 KST m=+0.000021800 start
2022-08-21 17:01:55.184932851 +0900 KST m=+1.000210473 done
2022-08-21 17:01:55.18507731 +0900 KST m=+1.000354912 exiting...
```

- We first initialize an instance of WaitGroup `wg`. 

- Then we add 1 to `wg` because we want to wait for one goroutine to finish. 

- We then run the goroutine. Inside the goroutine, we make a `defer` call to `wg.Done()` to make sure that we decrement the number of goroutines to wait on. If we dont' do this, then the code will wait forever for the goroutine to finish and will result in a deadlock.

- After the goroutine call, we make sure to block the code until the WaitGroup is empty. We do this by calling `wg.Wait()`.

### Why use WaitGroups over channels?

Now that we know how to use WaitGroups, a natural flow of thought leads us to this question: why use WaitGroups over channels?

Based on my experience, there are a few reasons.

- WaitGroups tend to be more intuitive. When you read a snippet of code, when you see a WaitGroup, you immediately know what the code is doing. The method names are explicit and to the point. However, it's sometimes not that clear with channels. Using channels is clever, but it can be a hassle to understand when you are reading a complicated piece of code.

- There are times where you don't really need to use a channel. For example, let's look at this code:
  
  ```go
  var wg sync.WaitGroup
  
  for i := 0; i < 5; i++ {
  	wg.Add(1)
  	go func() {
  		defer wg.Done()
  
  		fmt.Println(time.Now(), "start")
  		time.Sleep(time.Second)
  		fmt.Println(time.Now(), "done")
  	}()
  }
  
  wg.Wait()
  fmt.Println(time.Now(), "exiting...")
  ```
  
  You can see that the goroutine isn't communicating data to other goroutines. If your goroutines are one-off jobs where you don't need to know the results, using a WaitGroup is desirable. Now take a look at this code:
  
  ```go
  ch := make(chan int)
  
  for i := 0; i < 5; i++ {
  	go func() {
  		randomInt := rand.Intn(10)
  		ch <- randomInt
  	}()
  }
  
  for i := 0; i < 5; i++ {
  	fmt.Println(<-ch)
  }
  ```
  
  Here, the goroutine is sending data back to the channel. In these cases, we don't need to use a WaitGroup because it would be redundant. Why wait for goroutines to finish if receiving already does enough blocking?

WaitGroups are specialized for dealing with waiting for goroutines. I feel like a channel's main purpose is to communicate data. You can't use a WaitGroup to send and receive data, but you can use a channel to sync your goroutines.

In the end, there is no right answer. I know that this can be annoying, but it really is up to you and the team you are working with. Whatever works is best, and no answer is wrong. My personal preference is to use WaitGroups for syncing, but your mileage may vary. Pick something that is the most intuitive to you.

### One thing to watch out for

Sometimes, you might need to pass the WaitGroup instance to the goroutine. There may be several WaitGroups that handle different goroutines, or maybe it's a design choice. Whatever the reason may be, make sure to pass the pointer to the WaitGroup, like so:

```go
var wg sync.WaitGroup

for i := 0; i < 5; i++ {
	wg.Add(1)
	go func(wg *sync.WaitGroup) {
		defer wg.Done()

		fmt.Println(time.Now(), "start")
		time.Sleep(time.Second)
		fmt.Println(time.Now(), "done")
	}(&wg)
}

wg.Wait()
fmt.Println(time.Now(), "exiting...")
```

The reason is that Go is a pass-by-value language. What this means is that whenever you pass an argument to a function, Go will actually make a copy of the argument and pass that instead of the original object. What happens in this context is that the entire WaitGroup object will be copied, meaning that the goroutine would be dealing with a completely different WaitGroup. The `wg.Done()` won't subtract from the original `wg`, but a copy of it which only lives inside the goroutine.

### Conclusion

I hope this guide helped you design your concurrent code easily! WaitGroups are easy to understand and implement. Next time, we will talk about how to listen to multiple channels at the same time. See you next week!

You can also read this post on Medium and Dev.to.
