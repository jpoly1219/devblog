---
title: "Debugging Go Code With Delve"
date: 2022-06-19T10:41:24+09:00
draft: true
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

### Going over the commands I use most often

This is the code we will use for this example.

```go

```

Let's start by, well, running Delve. `cd` into the directory that includes your `main.go` file, then run this:

```
$ dlv debug
```

If you are debugging tests, you can `cd` into the directory that holds your `*_test.go` files, then run this:

```
$ dlv test
```

As far as I know, there isn't a GUI frontend for Delve, so you need to use the command line to use Delve. This is fine because Delve has a friendly CLI that you can actually understand.

```
$ 
```



### Case in point: Memory bug

blah

### Conclusion

concl
