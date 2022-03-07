---
title: "Watch Out for These Tricky Things in Go"
date: 2022-03-06T20:08:30+09:00
draft: false
---

I love Go. I wrote an entire blog post about why I love the language. It is an attractive language with many fun features. Go is considered a relatively easy language, but sometimes it can be tricky. I certainly had my share of challenges that I had to do a lot of research on while learning the language. In this post, I listed some of the things that bummed me out when I was learning Go so that you don't have to struggle like me. Hope you enjoy it!

### You can't compare certain data types.

Coming from Python, I wasn't expecting to have issues with comparing objects and values. However, it turns out that there are certain data types that you cannot compare literally in Go. These include functions, maps, and slices. 

```go
package main

import "fmt"

func main() {
    slice1 := []int{0, 1, 2}
    slice2 := []int{0, 1, 2}
    if slice1 == slice2 {
        fmt.Println("equal")
    }
}
```

You would expect this to print `equal`, but it does this instead:

```
invalid operation: slice1 == slice2 (slice can only be compared to nil)
```

In Go, unless you are directly comparing to `nil`, you cannot compare functions, maps, or slices for equality.

There aren't a lot of cases where you would directly compare functions though, which makes it less of a hassle. If you must, you should try to compare the returned values from each function. The problem arises when you are trying to compare maps or slices. Comparing them, especially slices, tend to happen a lot more than you think.

When you have to compare non-comparable data types, there are three different ways to do so. One is to iterate through all of the items and see if they match the other. 

```go
package main

import "fmt"

func main() {
    slice1 := []int{0, 1, 2}
    slice2 := []int{0, 1, 2}
    for i := range slice1 {
        if slice1[i] != slice2[i] {
            fmt.Println("not equal")
            break
        }
    }
    fmt.Println("equal")
}
```

This simple solution is slightly tedious and can get out of hand as the data gets larger. A better option would be to use Go's `reflect` package. This package has a nifty function called `DeepEqual`, which can compare two objects and see if they are deeply equal. 

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    slice1 := []int{0, 1, 2}
    slice2 := []int{0, 1, 2}

    ok := reflect.DeepEqual(slice1, slice2)
    if ok {
        fmt.Println("equal")
    } else {
        fmt.Println("not equal")
    }
}
```

Normally, `reflect.DeepEqual` will be fine, but there are certain data types that it cannot compare, such as `time.Time`. Currently, the most recommended way to compare objects is to use the `cmp` package from Google.

```go
package main

import (
    "fmt"

    "github.com/google/go-cmp/cmp"
)

func main() {
    slice1 := []int{0, 1, 2}
    slice2 := []int{0, 1, 2}

    ok := cmp.Equal(slice1, slice2)
    if ok {
        fmt.Println("equal")
    } else {
        fmt.Println("not equal")
    }
}
```

Using `cmp.Equal` is the best option because you can also pass in options to allow certain fields or ignore certain data types, allowing for more granular control. The package also comes with a lot of useful methods you might need when comparing two values.

### You can't return a nil value for everything.

A common pattern you will see in Go is functions returning two values: one object and one error object. For example, take a look at this code.

```go
func LinesFromFile(r io.Reader) ([]string, error) {
    var lines []string
    scanner := bufio.NewScanner(r)
    for scanner.Scan() {
        lines = append(lines, scanner.Text())
    }
    if err := scanner.Err(); err != nil {
        return nil, err
    }
    return lines, nil
}
```

In short, this code scans line by line, appends each line into a slice of string, then returns the slice and an error object. If nothing goes wrong, the function returns a slice of string with `nil` as its error. If an error occurs, however, it returns `nil` for the slice and a new error object. This type of return is commonly used in Go.

The problem is that you cannot always return `nil`. In Go, `nil` only represents uninitialized zero values for pointers, channels, interfaces, slices, maps, and functions. The two most popular data types, int, and string cannot be set as nil. The zero state of an int is `0`, and the zero state of a string is `""`.

So how would you return nil in this case? Of course, you could just return `0` or `""`, and catch it. You usually don't even have to check this, because a well-written Go code will have an explicit error check after a function call. However, there are cases where a function will not return an error object, or you must require a `nil` value. In this case, you can just return a pointer to an int or a string instead.

### You can't iterate over maps reliably.

This one really tripped me when I first tried to do it. Here's an example:

```go
package main

import "fmt"

func main() {
    people := map[string]int{"Alice": 10, "Bob": 20, "Charlie": 30}
    for key, value := range people {
        fmt.Println(key, value)
    }
}
```

You would expect to see an output like this:

```
Alice 10
Bob 20
Charlie 30
```

But try running this a couple of times, and you will notice something odd.

```
Charlie 30
Alice 10
Bob 20
```

Wait, what? Why did the order change? Well, in Go, iteration over maps is *unspecified* and *unstable*. It is unspecified because the order of keys will be random, and it is unstable because every run of the program will result in different outcomes. This is an intended design by the Go developers to prevent the developers from relying on it. Developers started to rely on the stable iteration order, which led to portability issues as the language developed and the iteration order altered slightly.

This may have been a good design choice, but it doesn't change the fact that we need a way to reliably iterate over maps. To do this, maintain a sorted array or slice of the map keys. Iterate over this instead.

```go
package main

import "fmt"

func main() {
    people := map[string]int{"Alice": 10, "Bob": 20, "Charlie": 30}
    keySlice := []string{"Alice", "Bob", "Charlie"}
    for _, value := range keySlice {
        fmt.Println(value, people[value])
    }
}
```

This will always make sure the values are printed in the order of keys inside `keySlice`.

### Conclusion

I am planning on writing part 2 for this series. I think the best way to describe Go to newcomers is that it is a relatively easy language, but has its quirks due to it being designed differently from other popular languages such as Python and Javascript. Nonetheless, these quirks are all overcomeable with some experience and practice, so don't let these discourage you from learning Go!

Thank you for reading! You can also read this post on [Dev.to]([Watch Out for These Tricky Things in Go - DEV Community](https://dev.to/jpoly1219/watch-out-for-these-tricky-things-in-go-5bkn)) and [Medium](https://medium.com/@jpoly1219/watch-out-for-these-tricky-things-in-go-a1109f68549c).


