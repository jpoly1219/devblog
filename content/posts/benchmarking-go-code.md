---
title: "Benchmarking Go Code"
date: 2022-06-26T13:53:28+09:00
draft: false
---

If you have been programming in Go for some time, you probably really like the tooling that comes with it. The language itself is packed to the brim with useful tooling such as `go test`, `go fmt`, `go build`, etc. We have gone over testing before. Today, we will be looking at how we can benchmark our Go code.

### Why you should run benchmarks

Ever wondered how fast and efficient your code runs? Benchmarks are the most straightforward way to check the performance of your code. The benchmark tool provided by Go focuses on these aspects:

- Time taken per operation

- Memory allocated per operation

Benchmarks aren't a must, but they can be a good way to check for bottlenecks. Also, running numbers is fun. If you are a content creator, you probably check your feed daily, keeping track of how many new views your recent posts got. Benchmarks give me a similar sense of joy. 

### Insertion Sort vs. Merge Sort

We will use the following code for this tutorial.

```go
package main

import (
    "fmt"
)

func main() {
    list := []int{3, 4, 1, 5, 2}
    iList := insertionSort(list)
    mList := mergeSort(list)
    fmt.Println(iList, mList)
}

func insertionSort(list []int) []int {
    for i, num := range list {
        j := i - 1

        for j >= 0 && num < list[j] {
            list[j+1] = list[j]
            j -= 1
        }
        list[j+1] = num
    }

    return list
}

func mergeSort(list []int) []int {
    if len(list) > 1 {
        mid := len(list) / 2
        left := list[:mid]
        right := list[mid:]

        mergeSort(left)
        mergeSort(right)

        i, j, k := 0, 0, 0

        for i < len(left) && j < len(right) {
            if left[i] < right[j] {
                list[k] = left[i]
                i++
            } else {
                list[k] = right[j]
                j++
            }
            k++
        }

        for i < len(left) {
            list[k] = left[i]
            i++
            k++
        }

        for j < len(right) {
            list[k] = right[j]
            j++
            k++
        }
    }

    return list
}
```

The above is an implementation of two popular sorting algorithms: insertion sort and merge sort.

An insertion sort is a simple sorting algorithm that works like this:

- Iterate through the list from the beginning.

- If the current number is smaller than the previous number, move it to the front until it becomes bigger than the previous number.

Merge sort takes a different approach:

- Divide the list in half, so that you end up with a left list and a right list. Do this recursively on the left list and the right list.

- When you can't divide anymore, merge the left list and right list in ascending order.

- Merge the lists until you end up with the sorted original list.

These diagrams will help you understand the two sorts better.

- Insertion Sort:

![](https://media.geeksforgeeks.org/wp-content/uploads/insertion_sort-recursion.png)

https://media.geeksforgeeks.org/wp-content/uploads/insertion_sort-recursion.png

- Merge Sort:

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/Merge_sort_algorithm_diagram.svg/1064px-Merge_sort_algorithm_diagram.svg.png)

https://en.wikipedia.org/wiki/Merge_sort#/media/File:Merge_sort_algorithm_diagram.svg

### Benchmarking the code

Let's write our benchmark code.

```go
package main

import (
    "fmt"
    "math/rand"
    "testing"
)

func BenchmarkInsertionSort(b *testing.B) {
    inputSize := []int{10, 100, 1000, 10000, 100000}
    for _, size := range inputSize {
        b.Run(fmt.Sprintf("input_size_%d", size), func(b *testing.B) {
            testList := make([]int, size)
            for i := 0; i < size; i++ {
                testList[i] = rand.Intn(size)
            }
            b.ResetTimer()

            for i := 0; i < b.N; i++ {
                insertionSort(testList)
            }
        })
    }
}

func BenchmarkMergeSort(b *testing.B) {
    inputSize := []int{10, 100, 1000, 10000, 100000}
    for _, size := range inputSize {
        b.Run(fmt.Sprintf("input_size_%d", size), func(b *testing.B) {
            testList := make([]int, size)
            for i := 0; i < size; i++ {
                testList[i] = rand.Intn(size)
            }
            b.ResetTimer()

            for i := 0; i < b.N; i++ {
                mergeSort(testList)
            }
        })
    }
}
```

All benchmark codes are named in the format of `BenchnmarkXxx`, where `Xxx` is the name of the function being benchmarked starting with a capital letter.

We are going to test the two sorting algorithms using different input sizes. These are defined inside `inputSize`. For each input size, we launch a `b.Run` which fires off a single benchmark. `b.Run` accepts two parameters: first is the name of the benchmark and the second is the actual benchmark function to run. `fmt.Sprintf("input_size_%d", size)` is a good way to dynamically name our benchmarks, as it will name our benchmarks like so: `input_size_10`, `input_size_100`, and so on.

You could make one benchmark function for each input size, but the code will become unnecessarily long, so using `b.Run` is the idiomatic approach.

```go
testList := make([]int, size)
for i := 0; i < size; i++ {
    testList[i] = rand.Intn(size)
}
b.ResetTimer()
```

This portion populates our test data. We create a slice of size `size`, and populate it with random integers. We call `b.ResetTimer` in the end to reset our benchmark timer. `b.Run` starts the timer at the time of launching it, so it will by default include the time taken for initializing data. This is misleading, and we reset the timer before running the benchmark.

```go
for i := 0; i < b.N; i++ {
    mergeSort(testList)
}
```

This is the loop that runs our `mergeSort` function. The loop runs `b.N` time, which is an arbitrary number calculated to yield stable and reliable results. It's like running a lab multiple times to find the average data of different data points to make the results more precise.

Let's run the benchmark now.

```
$ go test -bench=. -benchmem
goos: linux
goarch: amd64
pkg: example.com/benchmarking
cpu: Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz
BenchmarkInsertionSort/input_size_10-8          127217533                9.369 ns/op           0 B/op          0 allocs/op
BenchmarkInsertionSort/input_size_100-8         13812344                89.95 ns/op            0 B/op          0 allocs/op
BenchmarkInsertionSort/input_size_1000-8         1508656               743.6 ns/op             0 B/op          0 allocs/op
BenchmarkInsertionSort/input_size_10000-8         158587              7365 ns/op               0 B/op          0 allocs/op
BenchmarkInsertionSort/input_size_100000-8             1        1296239500 ns/op               0 B/op          0 allocs/op
BenchmarkMergeSort/input_size_10-8              14895802                76.45 ns/op            0 B/op          0 allocs/op
BenchmarkMergeSort/input_size_100-8              1000000              1045 ns/op               0 B/op          0 allocs/op
BenchmarkMergeSort/input_size_1000-8               89268             14178 ns/op               0 B/op          0 allocs/op
BenchmarkMergeSort/input_size_10000-8               7522            156233 ns/op               0 B/op          0 allocs/op
BenchmarkMergeSort/input_size_100000-8               678           1743882 ns/op               0 B/op          0 allocs/op
PASS
ok      example.com/benchmarking        15.215s
```

We run `go test` like how we would run tests. Then we pass these two flags:

- `-bench` specifies which benchmark function to run. To run all of them, pass `-bench=.`.

- `-benchmem` shows you how much memory is allocated per operation, and how many allocations occur.

To the left, you can see the names of the benchmark functions. The `-8` at the end means that we are using 8 cores of our CPU.

Next to that is a huge number that decreases as our input size increases. These are your `b.N`, or how many times the benchmark has run your function. We call this the number of operations.

To the right of that, you can see the average time taken per operation. This means that for every run of the function, approximately x nanoseconds passed.

The last two columns show the average amount of memory bytes used for each operation, and the number of allocations per operation.

You can see that for smaller input sizes of 10,000 and below, the insertion sort is more efficient. However, from that point onwards, merge sort is much more efficient.

Both are memory efficient because they both do not require much allocation of memory. In Go, taking slices doesn't cost memory. More on that [here]([pointers - Slices in golang do not allocate any memory? - Stack Overflow](https://stackoverflow.com/questions/29196475/slices-in-golang-do-not-allocate-any-memory#:~:text=Because%20slices%20are%20multiword%20structures,and%20length%20pairs%20in%20C.)).

### Conclusion

Benchmarks are a useful tool to check if your code is running efficiently. This leaves us with one question: this means that we don't need to benchmark our code if we write a well-optimized code from the get-go, right? In a perfect world, this is true. However, I'd advise against it, as countless other people have.

> Premature optimization is the root of all evil.
> 
> --<cite>Donald Knuth, the legendary computer scientist, and mathematician</cite>

Focus on writing readable, working code. Then run a benchmark if you start running into bottlenecks. Maintainability is more important than performance in many cases, especially when modern computers are so fast.

Thank you for reading! You can also read this post on [Medium](https://medium.com/@jpoly1219/benchmarking-go-code-7c780e29a7f) and [Dev.to](https://dev.to/jpoly1219/benchmarking-go-code-1k23).
