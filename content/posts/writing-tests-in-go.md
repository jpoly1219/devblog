---
title: "Writing Tests in Go"
date: 2022-06-12T16:54:43+09:00
draft: false
---

In today's post, we will be looking into how to write tests in Go. Test-driven development, also known as TDD, is a paradigm in which developers are encouraged to test their code as they go.

### What is a test and why should I care?

![](/writing-tests-in-go.png)

A test is a piece of code written to verify that your code behaves as it should. At first, this may sound quite tedious.

> Meh. I know my code through and through. I know what I'm doing. Tests are such a waste of time.
> 
> -- <cite>Famous last words by yours truly :')</cite>

I used to think that writing tests were a waste of time as well. I wrote the code, so I should know best if it works or not, right? I can always just print debug and fix the bugs, right? There is no way that my function will bug out when it is called from somewhere else, right? ...Right?

There are a couple of reasons why writing tests are important. When you advance through your career or your school program, chances are that you will be working with other people. When you are working on a project together, you may be responsible for writing a piece of code that must be used by other people. We call this a *library*. When other people depend on your code working as it should, it is crucial to make sure that you deal with edge cases, or else they need to wait around until you fix it. This can be very time-consuming, and can mostly be prevented by testing your code on the way. Tests are like tools that you didn't know you need when coding solo, but happens to be a lifesaver in team projects.

A second reason to write tests is that it often leads to cleaner code. When we refer to tests, we usually mean *unit tests*, which means that we are testing each individual component of the code. In order to write these tests, you need to write your code in a way that is modular. For example, instead of one gigantic, bloated function, we can take that apart into atomic functions that do one thing well. Then each of those functions can be tested individually to ensure that they work properly.

A third reason is more subjective. I think TDD helps me visualize things better. In TDD, you are encouraged to write your tests first before actually implementing the functionality. In my case, by writing the test first, I am more sure of what my module should accept as input and what to spit out as output. I can also think of edge cases beforehand so that I don't need to go and fix my implementation afterward.

### Writing tests in Go

It is pretty easy to write tests in Go. Go includes the `testing` package in its standard library, which I think is a very thoughtful decision from the developers of the language. You don't really need a separate testing framework or libraries like other languages, which I appreciate.

Go tests are stored in `*_test.go` files. The asterisk stands for your `.go` file with functions that you'd like to test. For example, let's say that I have a `areas.go` file that contains functions that calculate areas of different shapes. The tests for those functions will be in `areas_test.go`.

Let's begin with a high-level overview of our code. As I mentioned previously, `areas.go` will store three functions that calculate areas: `findTriangleArea`, `findSquareArea`, and `findCircleArea`. We will write the implementations later. We should write the input and output first, or else Go will complain when we are writing our tests.

```go
package areas

func findTriangleArea(base, height float64) float64 {
    return 0.0
}

func findSquareArea()

func findCircleArea()
```

We can now start writing our tests. Remember, we want to write the tests before the functions. Create a file named `areas_test.go` and type the following:

```go
package areas

import (
    "testing"
)
```

We start by importing these packages.

```go
func TestFindTriangleArea(t *testing.T) {
    base := 2.0
    height := 3.0
    expectedArea := 3.0
    actualArea := findTriangleArea(base, height)

    if actualArea != expectedArea {
        t.Fatalf("expected %v, got %v", expectedArea, actualArea)
    }
}
```

This is the test function for `findTriangleArea`. All Go test functions have names that are in the format of `TestXxx(t *testing.T)`, where `Xxx` is the name of the function that you want to test. The first character of your test must be capitalized. `testing.T` is a variable that is responsible for keeping track of the state of your test. `T` can be used to call specific methods such as `Errorf` or `Fatalf` to terminate tests, return errors, log results, and more.

Inside the function, we see the main logic of the test. Simply put, a test checks to see if the expected value and the actual value match. In our case, we call `findTriangleArea` with a base of 2.0 and a height of 3.0. We expect to see an output of 3.0 because a triangle's area is found by `(base * height) / 2`. If the call to `findTriangleArea` returns a non-nil error, or if the returned area does not match the expected area, we will output an error and stop the test.

`t.Fatalf` is like `fmt.Sprintf` in the sense that we can format the output to our liking. I like to output the expected value, the actual value we got, and any errors if it's there.

Let's run the test. To run a test in Go, we go to the terminal, and `cd` into the directory where the test file is. Then, we run `go test -v`. You will get an output like this:

```
$ go test -v
=== RUN   TestFindTriangleArea
    areas_test.go:15: expected 3, got 0
--- FAIL: TestFindTriangleArea (0.00s)
FAIL
exit status 1
FAIL    example.com/areas     0.002s
```

Now we can write out the implementation of `findTriangleArea`.

```go
func findTriangleArea(base, height float64) float64 {
    area := base * height / 2
    return area
}
```

Simple, right? I'll write one for other functions as well. Here's our final result.

```go
// areas.go
package areas

import "math"

func findTriangleArea(base, height float64) float64 {
    area := base * height / 2
    return area, nil
}

func findSquareArea(side float64) float64 {
    area := side * side
    return area
}

func findCircleArea(radius float64) float64 {
    area := math.Pi * radius * radius

    // round to the 3rd decimal place
    roundedArea := math.Round(area*1000) / 1000
    return roundedArea
}
```

```go
package areas

import (
    "testing"
)

func TestFindTriangleArea(t *testing.T) {
    base := 2.0
    height := 3.0
    expectedArea := 3.0
    actualArea := findTriangleArea(base, height)

    if actualArea != expectedArea {
        t.Fatalf("expected %v, got %v", expectedArea, actualArea)
    }
}

func TestFindSquareArea(t *testing.T) {
    side := 2.0
    expectedArea := 4.0
    actualArea := findSquareArea(side)

    if actualArea != expectedArea {
        t.Fatalf("expected %v, got %v", expectedArea, actualArea)
    }
}

func TestFindCircleArea(t *testing.T) {
    radius := 2.0
    expectedArea := 12.566
    actualArea := findTriangleArea(radius)

    if actualArea != expectedArea {
        t.Fatalf("expected %v, got %v", expectedArea, actualArea)
    }
}
```

If we now run `go test -v`, we will get the following result:

```
$ go test -v
=== RUN   TestFindTriangleArea
--- PASS: TestFindTriangleArea (0.00s)
=== RUN   TestFindSquareArea
--- PASS: TestFindSquareArea (0.00s)
=== RUN   TestFindCircleArea
--- PASS: TestFindCircleArea (0.00s)
PASS
ok    example.com/areas     0.001s
```

Awesome! All our test passes.

### Testing for multiple cases

This is all sunshine and rainbows, but what if we want to test for multiple cases? There are probably more than one cases that you want to run the test on. You'd be right.

Intuitively, we could just write something like this:

```go
func TestFindSquareArea(t *testing.T) {
    side := 2.0
    expectedArea := 4.0
    actualArea := findSquareArea(side)

    if actualArea != expectedArea {
        t.Fatalf("expected %v, got %v", expectedArea, actualArea)
    }

    side = 3.0
    expectedArea = 4.0
    actualArea := findSquareArea(side)

    if actualArea != expectedArea {
        t.Fatalf("expected %v, got %v", expectedArea, actualArea)
    }

    // repeat...
}
```

Because this code will get ugly real fast, let's try something else.

```go
func TestFindSquareArea(t *testing.T) {
    type findSquareAreaTest struct {
        side float64
        expected float64
    }

    findSquareAreaTests := []findSquareAreaTest{
        {2.0, 4.0},
        {3.0, 9.0},
        {4.0, 16.0},
        {0.0, 0.0},
    }

    for _, test := range findSquareAreaTests {
        output := findSquareArea(test.side)
        if output != test.expected {
            t.Fatalf("expected %v, got %v", test.expected, output)
        }
    }
}
```

This looks much better. We declared a struct `findSquareAreaTest` that holds the `side` value and the `expected` value. Then, we create a slice of that struct, which holds all of our test cases. We just need to use a for loop to iterate through each test case. This type of approach is called *table-driven testing* because we are using a table that holds many test cases.

### Conclusion

Thank you for reading this post. I hope you found it useful. There are certainly more advanced testing strategies than what I've listed here. I highly recommend you check out the [documentation for the `testing` package](https://pkg.go.dev/testing) to dive deeper. However, this post will get you started with test-driven development. Try applying it in your own personal project! I'll see you next time with more interesting topics. Bye!

You can also read this post on Medium and Dev.to.
