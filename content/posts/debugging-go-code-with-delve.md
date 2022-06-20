---
title: "Debugging Go Code With Delve"
date: 2022-06-19T10:41:24+09:00
draft: false
---

> "Genius is 1% talent and 99% hard work."
> 
> "e = mc^2"
> 
> -- <cite>Albert Einstein</cite>

In the field of software development, the quotes can be changed slightly:

> "Software development is 1% programming and 99% debugging."
> 
> "errors = more code ^ 2"
> 
> -- <cite>Some senior developer, probably</cite>

Debugging is something all developers must go through, and it does not care about your expertise. It's a frustrating process. To err is human, and errors will absolutely creep into your program. I don't know why, but errors are your spawns just like your code, so you are responsible for handling them.

### We all love to print debug...

If you are a computer science undergrad, then you probably had a bad experience with a project that doesn't seem to work the way you want it to. This would also apply to other developers as well. My code doesn't work, and I don't understand why.

I don't know why this is the case, but there is a universal tendency for developers to gravitate towards print debugging. At the part of the code where you assume the error to be, you would print the input and output variables.

Print debugging is often faster for fixing simple bugs. The most important part of debugging is to check whether you are feeding the right input to a function and whether the function is spitting out the desired output. Print statements can handle this easily. However, there are some nasty bugs that tend to manifest in one function but originate from a distant function. There are also cases where you may want to check the call stack, track the lifespan of a variable, etc.

Trust me, you will run into a situation where you feel like you are wasting a lot of time typing print statements. Your code will look ugly, and you will get lost in your own code. Those are the times when you wish you knew how to use a debugger.

If you aren't convinced, try thinking of it like this. It's good to know how to use debuggers. I think that using print debugging because you don't know how to use a debugger is bad practice. Like any skill, not knowing shouldn't be the main reason why you aren't using/learning it. That's like ordering takeout because you don't know how to cook. Or not working out because you don't know how to use the gym equipment. Or not studying because you don't understand a concept... you get the point.

### Using the Delve Debugger

There is a popular debugger called **gdb** that can debug Go code. However, the Go documentation does not recommend it and instead points you towards a better alternative: **Delve**.

Delve is like a toolbox that has a lot of tools to help you squash those nasty bugs. You don't have to catch a cockroach with your bare hands when you can use a bait station. Delve provides you with bug sprays, bait stations, torches, etc. Delve is used to catch Go bugs specifically, and it is quite easy to use.

First, let's install it. If you are running Go v1.16 or later (which you probably should), you can use the following line:

```
$ go install github.com/go-delve/delve/cmd/dlv@latest
```

Make sure you installed it by running this:

```
$ dlv version
```

Delve is used to debug the main package and tests. Unfortunately, it is rather limited in terms of debugging packages other than `main`, because Delve needs a working executable of the program, which requires a `main` package. Like me, if you are writing a library that does not have a `main` package, you need to write tests for your code, then debug that instead.

### Going over the commands I use most often, with an example code

This is the code we will use for this example.

```go
package main

import (
    "fmt"
    "math"
)

func main() {
    n1 := []float64{0.1, 0.2, 0.3, 0.4, 0.5}
    n2 := []float64{math.NaN(), 0.2, 0.3, 0.4, 0.5}

    mean1 := calcMean(n1)
    mean2 := calcMean(n2)

    fmt.Println("mean1:", mean1)
    fmt.Println("mean2:", mean2)
}

func calcMean(nums []float64) float64 {
    mean := 0.0
    for _, num := range nums {
        mean += num
    }
    mean /= float64(len(nums))

    return mean
}
```

The code is really simple! The `main` function defined two slices `n1` and `n2` and calls `calcMean` to calculate each mean value. This code looks fine, and the compiler isn't complaining. If there were any syntactical error, we won't even be able to compile our code. However, this code actually has one small issue. Can you guess what it might be?

Well, let's run the code and see what happens.

```
$ go run main.go
mean1: 0.3
mean2: NaN
```

Interesting. `mean1` seems alright, but `mean2` looks a bit weird. We want a number, not a `NaN` value. Why is this happening? Let's use the debugger to figure out what's actually going on.

`cd` into the directory that includes your `main.go` file, then run this:

```
$ dlv debug
```

If you are debugging tests, you can `cd` into the directory that holds your `*_test.go` files, then run this:

```
$ dlv test
```

As far as I know, there isn't a GUI frontend for Delve, so you need to use the command line to use Delve. This is fine because Delve has a friendly CLI that you can actually understand.

```
$ dlv debug
Type 'help' for list of commands.
(dlv) 
```

You will see that we have entered the Delve interface. Now we can run Delve-specific commands that will help us fix this bug.

```
(dlv) break main.go:8
Breakpoint 1 set at 0x496732 for main.main() ./main.go:8
```

The first command is `break`. It creates a *breakpoint* in your code. A breakpoint is a point in your code that you can stop the execution of. In this case, I set a breakpoint at line 8 in the file `main.go`. It is useful to set breakpoints before a suspicious code. Usually, it is a function that accepts an input and returns an output. To go to the breakpoint, we can use the following command:

```
(dlv) continue
> main.main() ./main.go:8 (hits goroutine(1):1 total:1) (PC: 0x496732)
     3: import (
     4:         "fmt"
     5:         "math"
     6: )
     7:
=>   8: func main() {
     9:         n1 := []float64{0.1, 0.2, 0.3, 0.4, 0.5}
    10:         n2 := []float64{math.NaN(), 0.2, 0.3, 0.4, 0.5}
    11:
    12:         mean1 := calcMean(n1)
    13:         mean2 := calcMean(n2)
```

`continue` will, well, continue traversing the code until the next breakpoint is reached. If there are no breakpoints set, `continue` will just run through the entire program, finishing the run. Here, continue will take us to line 8. Note that a breakpoint won't actually run the specified line. Let's go line by line now.

```
(dlv) next
> main.main() ./main.go:9 (PC: 0x496749)
     4:         "fmt"
     5:         "math"
     6: )
     7:
     8: func main() {
=>   9:         n1 := []float64{0.1, 0.2, 0.3, 0.4, 0.5}
    10:         n2 := []float64{math.NaN(), 0.2, 0.3, 0.4, 0.5}
    11:
    12:         mean1 := calcMean(n1)
    13:         mean2 := calcMean(n2)
    14:
(dlv) next
> main.main() ./main.go:10 (PC: 0x4967fa)
     5:         "math"
     6: )
     7:
     8: func main() {
     9:         n1 := []float64{0.1, 0.2, 0.3, 0.4, 0.5}
=>  10:         n2 := []float64{math.NaN(), 0.2, 0.3, 0.4, 0.5}
    11:
    12:         mean1 := calcMean(n1)
    13:         mean2 := calcMean(n2)
    14:
    15:         fmt.Println("mean1:", mean1)
```

We use the `next` command to go to the next line. Here, we traverse through lines 9 and 10, where we define `n1` and `n2`. Let's see how these slices are represented in our debugger.

```
(dlv) print n1
[]float64 len: 5, cap: 5, [0.1,0.2,0.3,0.4,0.5]
(dlv) print n2
Command failed: could not find symbol value for n2
```

We use the `print` command to see the details of a variable. We see something odd there, however. `print n1` works, but `print n2` doesn't. This is because when we are in a certain line, that line doesn't get executed until we move on to the next line. `n1` is defined in line 9, while `n2` is defined in line 10. We are currently at line 10, which means that line 9 has been executed, but line 10 has not yet been. To fix this, we just need to go to the next line.

```
(dlv) next
> main.main() ./main.go:12 (PC: 0x4968a5)
     7:
     8: func main() {
     9:         n1 := []float64{0.1, 0.2, 0.3, 0.4, 0.5}
    10:         n2 := []float64{math.NaN(), 0.2, 0.3, 0.4, 0.5}
    11:
=>  12:         mean1 := calcMean(n1)
    13:         mean2 := calcMean(n2)
    14:
    15:         fmt.Println("mean1:", mean1)
    16:         fmt.Println("mean2:", mean2)
    17: }
(dlv) print n1
[]float64 len: 5, cap: 5, [0.1,0.2,0.3,0.4,0.5]
(dlv) print n2
[]float64 len: 5, cap: 5, [NaN,0.2,0.3,0.4,0.5]
```

You can see how `print n2` works as expected now.

We are at line 12 at the moment, and this is where using a debugger can differentiate itself from print debugging. Delve has a feature where you can *step in* to a function. As its name suggests, stepping into the function allows us to go to the function and examine that function step by step. I'll show you what it means:

```
(dlv) step
> main.calcMean() ./main.go:19 (PC: 0x496ac0)
    14:
    15:         fmt.Println("mean1:", mean1)
    16:         fmt.Println("mean2:", mean2)
    17: }
    18:
=>  19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
    21:         for _, num := range nums {
    22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
```

Use `step` to step into a function. We can see how our scope changed from the `main` function to `calcMean` function. 

```
(dlv) args
nums = []float64 len: 5, cap: 5, [...]
~r0 = 0.000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004074232364696
```

Using `args` will show us the function parameters. `nums` is the input for `calcMean`. `~r0` denotes the return value of the function. `r` stands for return, and `0` stands for the 0th return value. If we have multiple return values, we will have something like `r0, r1, r2...`. The `~` represents an unnamed value. We know that `calcMean` will return a `float64`, but we didn't specify a name for it.

We can check function input by using the `print` command as well.

```
(dlv) print nums
[]float64 len: 5, cap: 5, [0.1,0.2,0.3,0.4,0.5]
```

So we know that there is nothing wrong with the input. Let's keep going. We want to keep track of the value of the `mean` variable. To do that, use the `display` command.

```
(dlv) display -a mean
0: mean = error could not find symbol value for mean
(dlv) next
> main.calcMean() ./main.go:20 (PC: 0x496ae5)
    15:         fmt.Println("mean1:", mean1)
    16:         fmt.Println("mean2:", mean2)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
=>  20:         mean := 0.0
    21:         for _, num := range nums {
    22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
0: mean = error could not find symbol value for mean
(dlv) next
> main.calcMean() ./main.go:21 (PC: 0x496aeb)
    16:         fmt.Println("mean2:", mean2)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
=>  21:         for _, num := range nums {
    22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
0: mean = 0
```

Using the `-a` flag will add variables to our display list. If we move on to the next line, you can see how the value of `mean` shows at the bottom.

There is a small error that says how we `could not find symbol value`. This is because `mean` is not defined yet. You can see how this error goes away when we are at line 21 and `mean` has been defined.

```
(dlv) next
> main.calcMean() ./main.go:22 (PC: 0x496b44)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
    21:         for _, num := range nums {
=>  22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
    27: }
0: mean = 0
(dlv) next
> main.calcMean() ./main.go:21 (PC: 0x496b56)
    16:         fmt.Println("mean2:", mean2)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
=>  21:         for _, num := range nums {
    22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
0: mean = 0.1
(dlv) next
> main.calcMean() ./main.go:22 (PC: 0x496b44)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
    21:         for _, num := range nums {
=>  22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
    27: }
0: mean = 0.1
(dlv) next
> main.calcMean() ./main.go:21 (PC: 0x496b56)
    16:         fmt.Println("mean2:", mean2)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
=>  21:         for _, num := range nums {
    22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
0: mean = 0.30000000000000004
(dlv) next
> main.calcMean() ./main.go:22 (PC: 0x496b44)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
    21:         for _, num := range nums {
=>  22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
    27: }
0: mean = 0.30000000000000004
(dlv) next
> main.calcMean() ./main.go:21 (PC: 0x496b56)
    16:         fmt.Println("mean2:", mean2)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
=>  21:         for _, num := range nums {
    22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
0: mean = 0.6000000000000001
(dlv) next
> main.calcMean() ./main.go:22 (PC: 0x496b44)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
    21:         for _, num := range nums {
=>  22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
    27: }
0: mean = 0.6000000000000001
(dlv) next
> main.calcMean() ./main.go:21 (PC: 0x496b56)
    16:         fmt.Println("mean2:", mean2)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
=>  21:         for _, num := range nums {
    22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
0: mean = 1
(dlv) next
> main.calcMean() ./main.go:22 (PC: 0x496b44)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
    21:         for _, num := range nums {
=>  22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
    27: }
0: mean = 1
(dlv) next
> main.calcMean() ./main.go:21 (PC: 0x496b56)
    16:         fmt.Println("mean2:", mean2)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
=>  21:         for _, num := range nums {
    22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
0: mean = 1.5
(dlv) next
> main.calcMean() ./main.go:24 (PC: 0x496b65)
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
    21:         for _, num := range nums {
    22:                 mean += num
    23:         }
=>  24:         mean /= float64(len(nums))
    25:
    26:         return mean
    27: }
0: mean = 1.5
```

I apologize for the long output, but I really wanted to show you how `mean` changes for every iteration. You can see that the mean is adding up.

```
(dlv) next
> main.calcMean() ./main.go:26 (PC: 0x496b87)
    21:         for _, num := range nums {
    22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
=>  26:         return mean
    27: }
0: mean = 0.3
```

`mean` is equal to 0.3 after the division, so the function works as expected for input of `n1`. Let's see what happens when we input `n2`.

```
(dlv) next
> main.main() ./main.go:12 (PC: 0x4968c5)
Values returned:
        ~r0: 0.3

     7:
     8: func main() {
     9:         n1 := []float64{0.1, 0.2, 0.3, 0.4, 0.5}
    10:         n2 := []float64{math.NaN(), 0.2, 0.3, 0.4, 0.5}
    11:
=>  12:         mean1 := calcMean(n1)
    13:         mean2 := calcMean(n2)
    14:
    15:         fmt.Println("mean1:", mean1)
    16:         fmt.Println("mean2:", mean2)
    17: }
0: mean = error could not find symbol value for mean
(dlv) display -d 0
(dlv)
```

We get out of the scope when we hit the return line. We need to remove `mean` from the display list because we are not tracking it anymore. Run `display -d` and follow it with an id of the variable in the list. In our case, it is 0.

```
(dlv) next
> main.main() ./main.go:13 (PC: 0x4968cb)
     8: func main() {
     9:         n1 := []float64{0.1, 0.2, 0.3, 0.4, 0.5}
    10:         n2 := []float64{math.NaN(), 0.2, 0.3, 0.4, 0.5}
    11:
    12:         mean1 := calcMean(n1)
=>  13:         mean2 := calcMean(n2)
    14:
    15:         fmt.Println("mean1:", mean1)
    16:         fmt.Println("mean2:", mean2)
    17: }
    18:
(dlv) step
> main.calcMean() ./main.go:19 (PC: 0x496ac0)
    14:
    15:         fmt.Println("mean1:", mean1)
    16:         fmt.Println("mean2:", mean2)
    17: }
    18:
=>  19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
    21:         for _, num := range nums {
    22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
(dlv) print nums
[]float64 len: 5, cap: 5, [NaN,0.2,0.3,0.4,0.5]
```

Again, we step into the `calcMean` function. We print `nums` and check that the input is as expected.

```
dlv) display -a mean
0: mean = error could not find symbol value for mean
(dlv) next
> main.calcMean() ./main.go:20 (PC: 0x496ae5)
    15:         fmt.Println("mean1:", mean1)
    16:         fmt.Println("mean2:", mean2)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
=>  20:         mean := 0.0
    21:         for _, num := range nums {
    22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
0: mean = error could not find symbol value for mean
(dlv) next
> main.calcMean() ./main.go:21 (PC: 0x496aeb)
    16:         fmt.Println("mean2:", mean2)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
=>  21:         for _, num := range nums {
    22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
0: mean = 0
```

We use the `display` command to keep track of `mean`.

```
(dlv) next
> main.calcMean() ./main.go:22 (PC: 0x496b44)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
    21:         for _, num := range nums {
=>  22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
    27: }
0: mean = 0
(dlv) next
> main.calcMean() ./main.go:21 (PC: 0x496b56)
    16:         fmt.Println("mean2:", mean2)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
=>  21:         for _, num := range nums {
    22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
0: mean = NaN
```

```
(dlv) next
> main.calcMean() ./main.go:22 (PC: 0x496b44)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
    21:         for _, num := range nums {
=>  22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
    27: }
0: mean = NaN
(dlv) next
> main.calcMean() ./main.go:21 (PC: 0x496b56)
    16:         fmt.Println("mean2:", mean2)
    17: }
    18:
    19: func calcMean(nums []float64) float64 {
    20:         mean := 0.0
=>  21:         for _, num := range nums {
    22:                 mean += num
    23:         }
    24:         mean /= float64(len(nums))
    25:
    26:         return mean
0: mean = NaN
```

And that, ladies and gentlemen, is where the bug is. Note how `mean` becomes `NaN` as soon as we add the first element of `nums`. And from that point on, `mean` stays `NaN` no matter what we add to it. This is, obviously, an undesirable behavior. We want to ignore NaN values and calculate the mean without them.

Exit the debugger by using `exit`.

Let's go back to our code and update the `calcMean` code to the following:

```go
func calcMean(nums []float64) float64 {
    mean := 0.0
    nanCount := 0
    for _, num := range nums {
        if math.IsNaN(num) {
            nanCount++
            continue
        }
        mean += num
    }
    mean /= float64(len(nums) - nanCount)

    return mean
}
```

If we encounter a NaN value, we increase `nanCount` by 1, and skip to the next iteration by using `continue`. When calculating the mean, we divide the sum of `nums` by the length of `nums` minus the number of NaNs.

Let's see how our code runs now.

```
$ go run main.go
mean1: 0.3
mean2: 0.35
```

Nice! We successfully debugged our code.

### Conclusion

We learned how to use Delve to debug our code like a pro. Of course, the above example could've been easily debugged by adding a print statement inside the loop in `calcMean`. However, when you encounter nasty bugs in your journey, you will thank yourself for reading this post and knowing how to use a debugger. Delve provides many more commands, which you can check out in their [documentation](https://github.com/go-delve/delve/tree/master/Documentation/cli). Last time, we learned how to write tests to prevent bugs. When bugs do appear, we now know how to use a debugger to squash them.

Thank you for reading! You can also read this post on [Medium](https://medium.com/@jpoly1219/debugging-go-code-with-delve-22fe9de7e380) and [Dev.to](https://dev.to/jpoly1219/debugging-go-code-with-delve-2l67).
