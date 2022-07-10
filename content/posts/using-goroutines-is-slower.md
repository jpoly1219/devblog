---
title: "Using Goroutines Is Slower"
date: 2022-07-10T15:36:45+09:00
draft: false
---

Ah, goroutines. One of the most defining features of the Go programming language. Once you understand the syntax of goroutines and the theory behind concurrency, you feel as if you just gained a superpower. A hammer, if you will. We get so excited to make everything concurrent. I am definitely guilty. I mean, why not, right? Concurrency solves the issue of blocking code, so making everything as concurrent as possible will speed things up, right?

Sometimes, too much is too much, and not everything is a nail that you can hammer.

### But first, an introduction to concurrency in Go

Reading this post, you probably have at least some experience writing concurrent Go code. But just in case, I will explain concurrency and goroutines quickly.

As we get better at programming and build bigger projects, we inevitably run into an obstacle. There is a job that takes at least a couple of seconds. Maybe that job is sending an email to your users. Maybe it is reading and parsing CSV or JSON. Maybe it's just a stupidly complicated calculation. It's a bit better if your program is meant to serve one or two users at a time. However, imagine having to send a million emails or having to parse a million JSON stream objects. Your service will be blocked by these slow operations, and people will have a horrible experience using it.

How do we solve this? A person thought about this question for some time and decided to think about it more after he cooked dinner. He wanted a nice, juicy grilled chicken. He started marinating the chicken and put it in the fridge to let it sit for 30 minutes. While the chicken was sitting in the fridge, he started chopping some lettuce and onions for his salad... then it came to him. This is what he needed to do! Just let the blocking code run first, and run other bits of the code in the meantime! He can just check on the chicken once it's done marinating, and grill it later!

The above story is a gross oversimplification of how concurrent programs work. Go uses goroutines to delegate these tasks. The main goroutine is responsible for running the `main` function, and the worker goroutines each handle parts of the code to run concurrently.

Now, concurrency is a bit different from parallelism, a similar concept. However, parallelism is like having two chefs cooking different things at the same time, while concurrency is like having one chef juggling different tasks. Yes, this might be confusing. I think the confusion stems from us treating each goroutine like separate objects. We call them *worker goroutines* to simplify them, but they aren't actually separate workers working together. They are merely *separate processes* that are fired off by a single chef. They are *goroutines*, not *goworkers* (Get it? Coroutines and coworkers? Yeah? ...Ok I'll stop).

### It's super effective!

We will use this snippet below for our experiment.

```go
package main

import (
    "encoding/csv"
    "fmt"
    "os"
)

func main() {
    db := map[string][][]string{
        "AgeDataset-V1.csv": nil,
        "neo_v2.csv":        nil,
        "nba.csv":           nil,
        "airquality.csv":    nil,
        "titanic.csv":       nil,
    }
    for file := range db {
        db[file] = ReadCsv(file)
    }
}

func ReadCsv(filepath string) [][]string {
    f, err := os.Open(filepath)
    if err != nil {
        fmt.Println(err)
    }
    defer f.Close()

    csvr := csv.NewReader(f)
    rows, err := csvr.ReadAll()
    if err != nil {
        fmt.Println(err)
    }

    return rows
}
```

`ReadCsv` is a very simple code used to read CSV files. It accepts the file's path and returns the data inside the file in a format of `[][]string`. The `main` function contains a `db` object which is a map that matches the file path to the data inside. This `main` function is serial because the for loop won't move onto the next iteration until the operation inside the loop is done.

Here is a concurrent version of the `main` function:

```go
func main() {
    db := map[string][][]string{
        "AgeDataset-V1.csv": nil,
        "neo_v2.csv":        nil,
        "nba.csv":           nil,
        "airquality.csv":    nil,
        "titanic.csv":       nil,
    }

    var wg sync.WaitGroup

    wg.Add(5)
    for file := range db {
        go func(file string) {
            defer wg.Done()
            db[file] = ReadCsv(file)
        }(file)
    }
    wg.Wait()
}
```

We first create `wg`, a waitgroup that keeps check of how many goroutines are still working. For each file, we fire off a goroutine that runs `ReadCsv` and subtracts 1 from `wg` once it's done. We wait at the end until every goroutine is done.

Our benchmark code is very simple:

```go
func BenchmarkMain(b *testing.B) {
    for i := 0; i < b.N; i++ {
        main()
    }
}
```

Here are the benchmark results:

```
// serial
$ go test -bench=Main -benchtime=10s
goos: linux
goarch: amd64
pkg: example.com/slowconc
cpu: Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
BenchmarkMain-8               18         609132683 ns/op
PASS
ok      example.com/slowconc    11.633s

// concurrent
$ go test -bench=Main -benchtime=10s
goos: linux
goarch: amd64
pkg: example.com/slowconc
cpu: Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
BenchmarkMain-8               20         559101265 ns/op
PASS
ok      example.com/slowconc    11.781s
```

You can see that the concurrent code ran about 50ms faster than the serial code. We are happy in these cases: concurrency does make our code run faster. Let's see what happens when this doesn't happen.

### It's not very effective...

Let's see when using goroutines backfires.

```go
func FindSum(list []int) int {
    sum := 0
    for _, number := range list {
        sum += number
    }
    return sum
}

func FindSumConc(list []int) int {
    sum := 0
    var rwm sync.RWMutex

    var wg sync.WaitGroup
    wg.Add(len(list))
    for _, number := range list {
        go func(number int) {
            defer wg.Done()
            rwm.Lock()
            defer rwm.Unlock()
            sum += number
        }(number)
    }
    wg.Wait()

    return sum
}
```

These are the two functions we will use to run our second experiment. Both of these return a sum of integers in a list. The concurrent version uses a mutex to prevent data race. This means that when one goroutine is writing to `sum`, no other goroutines can write to it.

We will use this benchmark code:

```go
func BenchmarkFindSum(b *testing.B) {
    list := make([]int, 0)
    for i := 0; i < 1000000; i++ {
        list = append(list, rand.Intn(1000000))
    }
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        FindSum(list)
    }
}

func BenchmarkFindSumConc(b *testing.B) {
    list := make([]int, 0)
    for i := 0; i < 1000000; i++ {
        list = append(list, rand.Intn(1000000))
    }
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        FindSumConc(list)
    }
}
```

Here are the results:

```
$ go test -bench=FindSum -benchtime=10s
goos: linux
goarch: amd64
pkg: example.com/slowconc
cpu: Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
BenchmarkFindSum-8                 50497            243623 ns/op
BenchmarkFindSumConc-8                45         266713213 ns/op
PASS
ok      example.com/slowconc    27.144s
```

Surprising, isn't it? You'd assume that the concurrent version runs faster. After all, there are a million integers to add. But no, the concurrent version is about 1000 times slower.

### Why is this the case?

Simply put, it is because of the nature of the two problems. There are two main types of bottlenecks that a software developer encounters:

- **CPU-bound jobs.** These are jobs that are reliant on your CPU's speed. Jobs like complex calculation, breaking encryption, and finding the nth digit of pi are CPU-bound.

- **IO-bound jobs.** These are jobs that are reliant on read speeds and write speeds. Jobs like reading data from files, and requesting a resource over a network are IO-bound.

In our example, our first example of reading CSV files is IO-bound, because the goroutines have to call the operating system for reading files and wait until the data can be used. Our second example of calculating the sum is CPU-bound because we are constantly calculating until we are done with the last integer in the list.

There is a concept we need to understand when dealing with concurrency. When there are multiple goroutines running, our scheduler that manages the goroutines needs to decide which goroutine to run. Remember, concurrency isn't running several jobs simultaneously; it's about juggling between tasks. This act of juggling is called a **context switch.**

Here's an issue; when we context switch from one goroutine to another, the goroutine that got swapped out is technically paused for the time being. More exactly, they go into either of the two states: waiting or runnable. A waiting state means that the goroutine is waiting on a response from the system or the network. A runnable state means that the goroutine wants attention so that it can run whatever it was doing.

Let's think about this. When the goroutine is in a waiting state, the scheduler doesn't have to spend time worrying about it for now, because the goroutine has to wait for a response anyways. The momentary pause is therefore mostly fine. However, if the goroutine is in a running state, this means that the calculation is paused until the scheduler swaps it in. A momentary pause here would be devastating to performance.

If you need to throw and catch ten balls, it's an IO-bound task. You throw one ball, then move on to the next ball. You're not slowing down your progress when you switch, because when you throw a ball, that ball will need time to fly up and come back down anyways. You are basically throwing the rest of the balls during a downtime.

If you need to empty ten baskets of balls, however, it's a CPU-bound task. The only way to make this faster is to throw away the balls faster. Switching from one basket to the other stops any emptying from happening in other baskets. So here, switching doesn't help you finish any faster. If anything, the act of switching to another basket will add latency and slow you down.

IO-bound jobs benefit much more from concurrent designs than CPU-bound ones. We can see this in our benchmark results as well.

### Conclusion

Hopefully, this post wasn't too confusing. I remember getting absolutely lost when trying to study concurrency. If you ever wondered why your concurrent code runs slow, check if your job is CPU-bound or IO-bound. If it's CPU-bound, you may need to utilize parallelism instead. Or more simply, do some optimization and improve your algorithm's time complexity.

Thanks for reading! You can read this post on [Medium](https://medium.com/@jpoly1219/using-goroutines-is-slower-ba99c2fadce9) and [Dev.to](https://dev.to/jpoly1219/using-goroutines-is-slower-3b53).
