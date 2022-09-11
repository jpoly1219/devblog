---
title: "Timing Your Goroutines"
date: 2022-09-11T17:27:21-04:00
draft: true
---

Welcome back to the series. Today we will be tapping into the power of timers. Sometimes it makes more sense to wait for a certain amount of time than the amount of iterations. Go fortunately provides us with a couple of ways to time our goroutines.

### Timer

The first way to time your goroutine is to use a timer. A timer is an object that lets us wait for a given time. This is how you use a timer in Go.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println(time.Now())
	myTimer := time.NewTimer(time.Second)

	fmt.Println(<-myTimer.C)
}
```

```
2022-09-11 18:07:16.4523706 -0400 EDT m=+0.000035801
2022-09-11 18:07:17.4527271 -0400 EDT m=+1.000392201
```

You can see how timers use a channel to send messages. Once we receive the message, then we can move onto the next line of code. Remember how channels block until we can receive from them.

As shown above, timers don't require goroutines to run. You could use a timer in a sequential code as well, but in that case, we would usually use `time.Sleep`. But hold on a second, we can use `time.Sleep` to wait for goroutines as well!

How is a timer different from `time.Sleep`? Well, timer uses a channel, so you can use it inside a select statement. You can't do that with `time.Sleep`. Imagine having to wait for a goroutine that is taking too long. A good way to time that goroutine out is to set a timeout timer to fire once a time limit is reached.

```go
func main() {
	timeout := time.NewTimer(time.Second * 5)

	resCh := make(chan string)
	go func() {
		time.Sleep(time.Second * 10)
		resCh <- "done!"
	}()

	select {
	case res := <-resCh:
		fmt.Println(res)
	case <-timeout.C:
		fmt.Println("timeout")
	}
}
```

```
timeout
```

You can also stop a timer manually in the middle. I haven't run into situations where I needed this capability, but I'm sure some of the readers can think of a creative way to use this.

```go
func main() {
	myTimer := time.NewTimer(time.Second * 5)

	go func() {
		<-myTimer.C
		fmt.Println("done")
	}()

	stop := myTimer.Stop()
	if stop {
		fmt.Println("stopped")
	}
}
```

```
stopped
```

That being said, if all you need to do is wait, then `time.Sleep` will be more efficient because your code doesn't need to create a separate channel.

### Ticker

Waiting for a goroutine to finish is one thing, but what if you want the goroutine to fire repeatedly? We could use a for loop to accomplish this.

```go
func main() {
	for i := 0; i < 5; i++ {
		fmt.Print(i, " ")
		go func() {
			fmt.Println("done")
		}()
		time.Sleep(time.Second)
	}
}
```

```
0 done
1 done
2 done
3 done
4 done
```

There is a more sophisticated way to implement this, and it is to use a ticker. A ticker sets a time interval between each ticks, and sends that signal to a channel.

```go
func main() {
	timer := time.NewTimer(time.Second * 5)
	ticker := time.NewTicker(time.Second)

	go func() {
		for i := range ticker.C {
			fmt.Println(i)
		}
	}()
	<-timer.C 
    fmt.Println("done")
}
```

```
2022-09-11 19:02:02.8421275 -0400 EDT m=+1.000275801
2022-09-11 19:02:03.8422471 -0400 EDT m=+2.000395301
2022-09-11 19:02:04.8424484 -0400 EDT m=+3.000596601
2022-09-11 19:02:05.8426019 -0400 EDT m=+4.000750101
2022-09-11 19:02:06.8427572 -0400 EDT m=+5.000905401 
done
```

You can also use a `select` statement with tickers, like how we used it with timers.

```go
func main() {
	ticker := time.NewTicker(time.Second)
	timer := time.NewTimer(time.Second * 3)
	var wg sync.WaitGroup

	wg.Add(1)
	go func() {
		defer wg.Done()
		for {
			select {
			case <-ticker.C:
				fmt.Println("ticking")
			case <-timer.C:
				return
			}
		}
	}()
	wg.Wait()

	fmt.Println("done")
}
```

```
ticking
ticking
ticking
done
```

You could also fire a new goroutine every tick.

```go
func main() {
	ticker := time.NewTicker(time.Second)
	timer := time.NewTimer(time.Second * 3)

	var wg sync.WaitGroup

	wg.Add(1)
	go func() {
		defer wg.Done()
		for {
			select {
			case <-ticker.C:
				go func() {
					fmt.Println("hi!")
				}()
			case <-timer.C:
				return
			}
		}
	}()
	wg.Wait()

	fmt.Println("done")
}
```

```
hi!
hi!
hi!
done
```

### Conclusion

The takeaway point here is that timers and tickers allow more complex behaviors in your program. You could implement a timeout functionality where you drop each request if it takes too long. You could write a code that sends automated messages to the server every second. The world is your oyster. I will see you guys next week with a post on how to let goroutines share data using mutexes. See you then!

You can also read this post on Medium and Dev.to. 
