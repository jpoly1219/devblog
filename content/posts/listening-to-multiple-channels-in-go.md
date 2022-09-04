---
title: "Listening to Multiple Channels in Go"
date: 2022-08-28T12:22:35-04:00
draft: true
---

Welcome back to the series! Today we will look at ways to listen to multiple channels at once. The guides before helped you get started with concurrency in Go. Although simple methods are often the best ones, you may have been stuck trying to implement more complex behaviors. After reading this guide, you will be able to make your concurrent code more flexible.

### The select keyword

We can use the `select` keyword to listen to multiple goroutines at once.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	c1 := make(chan string)
	c2 := make(chan string)

	go func() {
		time.Sleep(1 * time.Second)
		c1 <- time.Now().String()
	}()

	go func() {
		time.Sleep(2 * time.Second)
		c2 <- time.Now().String()
	}()

	for i := 0; i < 2; i++ {
		select {
		case res1 := <-c1:
			fmt.Println("from c1:", res1)
		case res2 := <-c2:
			fmt.Println("from c2:", res2)
		}
	}
}

```

```
from c1: 2022-09-04 14:30:39.4469184 -0400 EDT m=+1.000172801
from c2: 2022-09-04 14:30:40.4472699 -0400 EDT m=+2.000524401
```

The code above shows how the `select` keyword works.

- We first create two channels `c1` and `c2` to listen to.

- Then we spawn two goroutines that each sends the current time to `c1` and `c2`.

- Within the for loop, we create a `select` statement and define two cases: the first one for when we can receive from `c1` and the second one for when we can receive from `c2`.

You can see that the `select` statement is very similar in design to the `switch` statement. Both define different cases and run the respective code when a certain case is met. Also, we can see that the `select` statement is blocking. That is, it will wait until one of the cases is met.

We iterate twice for the loop because there are only two goroutines to listen to. More exactly, each goroutine is a fire-and-forget goroutine, meaning that they only send to a channel once before returning. Therefore, there are two messages maximum at all times in this code, and we only need to select twice.

### What if we don't know when jobs will end?

Sometimes we don't know how many jobs there are. In this case, put the `select` statement inside a while loop.

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	c1 := make(chan string)
	rand.Seed(time.Now().UnixNano())

	for i := 0; i < rand.Intn(10); i++ {
		go func() {
			time.Sleep(1 * time.Second)
			c1 <- time.Now().String()
		}()
	}

	for {
		select {
		case res1 := <-c1:
			fmt.Println("from c1:", res1)
		}
	}
}

```

Because we let a random number of goroutines run, we don't know how many jobs there are. Thankfully, the for loop at the bottom encasing the select statement will capture every output. Let's see what happens if we run this code.

```
from c1: 2022-09-04 14:48:47.5145341 -0400 EDT m=+1.000257801
from c1: 2022-09-04 14:48:47.5146126 -0400 EDT m=+1.000336201
from c1: 2022-09-04 14:48:47.5146364 -0400 EDT m=+1.000359901
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:
main.main()
        /home/jacob/blog/testing/listening-to-multiple-channels-in-go/main.go:22 +0x128
exit status 2
```

Hmm, the select statement received three times as expected, but the program errored out due to a deadlock. Why would this be the case?

Remember that without any condition, a for loop in Go will run forever. This means that the select statement will try to receive forever. However, there is a finite number of jobs to run. Even though there are no more jobs, the select statement will still try to receive.

Remember back in the first post of the series where I said that your program will run into a deadlock if you try to receive from an unbuffered channel when the sender is not ready? This is exactly the case in our example.

So how do we solve this? We can use a combination of the concepts covered in previous posts: exit channels and WaitGroups.

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

func main() {
	c1 := make(chan string)
	exit := make(chan struct{})
	rand.Seed(time.Now().UnixNano())
	var wg sync.WaitGroup

	go func() {
		numJob := rand.Intn(10)
		fmt.Println("number of jobs:", numJob)
		for i := 0; i < numJob; i++ {
			wg.Add(1)
			go func() {
				defer wg.Done()
				time.Sleep(1 * time.Second)
				c1 <- time.Now().String()
			}()
		}
		wg.Wait()
		close(exit)
	}()

	for {
		select {
		case res1 := <-c1:
			fmt.Println("from c1:", res1)
		case <-exit:
			return
		}
	}
}

```

```
3
from c1: 2022-09-04 15:09:08.6936976 -0400 EDT m=+1.000287801
from c1: 2022-09-04 15:09:08.6937788 -0400 EDT m=+1.000369101
from c1: 2022-09-04 15:09:08.6937949 -0400 EDT m=+1.000385101
```

- We create an exit channel and a WaitGroup.

- The number of jobs is random for every run. For `numJobs` amount of times, we fire off goroutines. To wait for the jobs to finish, we add them to `wg`. When a job is done, we subtract one from `wg`.

- Once all jobs are complete, we close the exit channel.

- We wrap the above section in a goroutine because we want all of it to be non-blocking. If we were to not wrap it in a goroutine, the `wg.Wait()` will wait until the jobs are done. This will block the code and won't let the for-select statement at the bottom run.

- Furthermore, because `c1` is an unbuffered channel, waiting for all the goroutines to send the message to `c1` will result in many messages being sent to `c1` without the for-select statement to receive them. This results in a deadlock because the receiver is not ready when the sender is.

### How to make select non-blocking

The `select` statement is blocking by default. How do we make this non-blocking? It's simple - we just add a default case.

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

func main() {
	ashleyMsg := make(chan string)
	brianMsg := make(chan string)
	exit := make(chan struct{})
	rand.Seed(time.Now().UnixNano())
	var wg sync.WaitGroup

	go func() {
		numJob := rand.Intn(10)
		fmt.Println("number of jobs:", numJob)
		for i := 0; i < numJob; i++ {
			wg.Add(2)
			go func() {
				defer wg.Done()
				time.Sleep(time.Duration(rand.Intn(10)) * time.Millisecond)
				ashleyMsg <- "hi"
			}()
			go func() {
				defer wg.Done()
				time.Sleep(time.Duration(rand.Intn(10)) * time.Millisecond)
				brianMsg <- "what's up"
			}()
		}
		wg.Wait()
		close(exit)
	}()

	for {
		select {
		case res1 := <-ashleyMsg:
			fmt.Println("ashley:", res1)
		case res2 := <-brianMsg:
			fmt.Println("brian:", res2)
		case <-exit:
			fmt.Println("chat ended")
			return
		default:
			fmt.Println("...")
			time.Sleep(time.Millisecond)
		}
	}
}

```

```
...
number of jobs: 4
brian: what's up
...
ashley: hi
...
...
brian: what's up
ashley: hi
ashley: hi
brian: what's up
...
...
ashley: hi
...
brian: what's up
...
chat ended
```

Aside from the lame conversation, we can see how a default case works. Instead of waiting for chats to arrive, we can do something when there are no channels to receive from. In this example, we just printed out ellipses, but you could do anything you'd like.

### Conclusion

That's it for this post! Now you can listen to multiple channels simultaneously, which can be a huge asset when you are developing your personal project. Thanks for reading, and I'll see you guys next time.

You can also read this post on Medium and Dev.to.
