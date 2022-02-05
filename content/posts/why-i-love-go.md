---
title: "Why I Love Go"
date: 2022-02-02T10:07:58+09:00
draft: true
---

# Why I Love Go

I am proud to say that I am a gopher. Out of all the languages under my belt, I probably have the least experience with Go, but I love it the most. Aside from the cute mascot, I think Go has some amazing features that made me fall in love with the language. I thought it would be fun to share my experiences and thoughts with you guys.

So without further ado, here are some reasons why I love Go.

### Code written in Go is very easy to read.

When I was first learning English, I thought reading comprehension was really hard. I for the love of God could not figure out what the passage was trying to say. Reading my own writing was hard enough. Reading other author's articles was a daunting challenge for me, who wasn't fluent in English.

The problem of comprehension is magnified in the realm of software development. When reading code, I think that two things are crucial for understanding a piece of code: a robust understanding of the syntax of the programming language the code is written in, and the ability to follow through the developer's thought process.

I guess code comprehension is more like solving a math problem. Remember back in school how the math teachers always told you to show your work? When I worked as a teacher's assistant, going through exam questions were pain in the ass because I sometimes couldn't follow the student's work. Reading other developer's code is exactly like that.

Go enforces a strict style guide. Check out the [Effective Go - The Go Programming Language](https://go.dev/doc/effective_go) page for more detailed information. The tl;dr is, Go encourages writing explicit, easy-to-follow code.

Having a strict style guide is important, because it allows the developers to write consistent code. All Go code have a consistent syntax, so anyone can get up to speed with the syntax. There are no real gotchas. When I was learning C++ and Javascript, every damn tutorial would tell me different ways to write a function or an iterator. Like, just why? How evil do you need to be to come up with four different ways to iterate through an array? And Javascript, why even bother providing both `==` and `===` ? Do I need a semi-colon or no? Why allow type-coercion when you *clearly* know it's a recipe for disaster? A lot of projects seem to have different style guidelines, and this can lead to developers writing inconsistent code. There is "your favorite way", "your company's favorite way", and five other ways that you should write for each open source side projects.

"But Jacob, Go code is redundant! There are no generics, so you need to write code for each type!" I understand your point. A downside of this enforced style guide is that it can sometimes lead to long codes. One of the main points that developers criticize Go for is that Go code tends to be very verbose and repetitive. For example, Go's way of explicitly handling errors can feel redundant at times. Also, the lack of generics can lead to developers feeling as if they are writing redundant code.

However, one thing that I want to emphasize is that being verbose isn't always bad. Of course, being unnecessarily verbose and repetitive is bad. However, if readability is not sacrificed and can even be improved by writing explicit code, I think the benefits that a couple more lines can bring are quite substantial.

### Static typing with type inference is a best of both worlds.

Static typing doesn't take that much effort while coding, but saves a lot of time when it comes to debugging. A small investment that pays huge dividends is always good. Static typing allows for more reliability, which is fitting for a backend, service-focused language. Imagine a server handling hundreds of thousands of users spewing out nonsense thanks to a runtime error. Imagine the countless hours and days you've spent staying up and trying to fix the bug, only to realize that one of the function was trying to concatenate an integer to a string. I haven't had a chance to develop a production system, but I know what it feels like to debug a runtime error that is caused by one of the most trivial errors. It's *really* annoying.

Static typing *can* get slightly annoying when there are just so many types to deal with. Check out C++ for example. When you are iterating through an array, you can either use good ol' `int`, or some other types like `size_t`, `iterator` or `auto`. I guess if you can keep track of all these types and know when to use each of them, then this approach gives more power to the developer. However, I think this bogs you down with the pressure of choice. Sometimes the abundance of types can get slightly irritating.

To counter this (or for those of you coming from dynamically typed languages), you can leverage Go's type inference to streamline your process. To borrow a quote from the Go documentation:

> It's a fast, statically typed, compiled language that feels like a dynamically typed, interpreted language. (go.dev/doc/)

### The way Go interfaces work is awesome.

I think there are two things that makes Go stand out from the rest of the programming languages. One is concurrency, and the other is the use of interfaces. These two are key features in Go, but many Go beginners tend to get confused by these concepts. I was no exception. Concurrency was better because I knew conceptually how a concurrent program should be designed, but interfaces? They were totally new. Coming from C++, Python and Javascript background, the idea of using interfaces and using a non-OOP language made me wonder why this was even a thing. Then one day, while refactoring some code in my personal project, I was trying to figure out how to make the code more DRY. After some time, the concept of interfaces struck a chord with me, and I love using interfaces ever since.

Interfaces are Go's answer to people who are used to the OOP paradigm. It is somewhat similar to inheritance and polymorphism in a sense that it allows the grouping of similar code, and allows for methods to take on different forms. Let me give you an example of how they differ.

In OOP, let's assume that three classes `Dog`, `Cat` and `Mouse` are inherited from the parent class `Animal`. If the `Animal` class has a method `makeSound()`, the children classes can implement the method differently and show different behaviors. For example, `Dog.makeSound()` can output `"woof"`, `Cat.makeSound()` can output `"meow"`, and `Mouse.makeSound()` can output `"squeak"`.

Now let's look at a Go example. In Go, since there are no classes, let's assume that there are three structs `Dog`, `Cat`, and `Mouse`. They are initially three separate entities. Because we want to give the ability for them to make sounds, we decide to assign each of them `makeSound()` method. Let's think about this for a second. If these structs all have `makeSound()` methods, then we can group these structs into a group named `Animal` than can `makeSound()`. In other words, the three structs satisfies the `Animal` interface, having all implemented `makeSound()`. So later on, if we have parts of our program that require animals to make sound, we don't have to write three different functions to accomodate for each animal struct. Instead, we can just let the function take the `Animal` interface as a parameter and call `makeSound()`.

Interitance and polymorphism in OOP tends to work from top-down. The problem with top down method that I've experienced is that it makes our brains rigid. The structure has to be formed before you can do anything. It is like organizing your files into a directory structure. You need to make the folders first before you can organize your files. Having to think ahead about the structure is a roadblock to flexible thinking. I feel like writing a parent class makes it so that my thought process must revolve around that parent class.

Interfaces work in an opposite way. They work from bottom up. You can just start writing variables, then notice a pattern emerge. It's like having a bunch of files, but then you start noticing how some can be grouped as images and some as videos. Then you create an abstraction layer images and videos, then organize the files. To me, this process is a lot more natural. This is how our brains are supposed to work - finding connections and patterns from its and bits of information.

I guess this point is more debatable. You may like the OOP way of creating a structure first, then writing your code. Or, like me, you may like the Go way of finding emerging patterns from your code and abstracting them as interfaces. I find Go's bottom-up approach to be a lot more natural and enjoyable.

### Go has an amazing standard library.

You can make a lot of stuff without relying on external libraries. I haven't had a chance to work in the industry yet, but I'm pretty sure there is a reason why a lot of people praise the standard library. Especially the `r/golang` subreddit.

This doesn't mean that you shouldn't use libraries. There are fantastic libraries that aid the development process such as `github.com/golang-jwt/jwt` or `github.com/gorilla/mux`. However, many things can be built using the standard library without heavily relying on frameworks.

Granted, the standard library isn't focused on certain areas such as GUI development or data science. I think this would be a pretty nice addition to the standard library, but neither seems to be the core purpose of Go, so it is understandable why these are omitted. However, the Go standard library is a joy to use for backend developers and DevOps engineers. Why would Go include libraries like `net/http` and `database/sql` in its standard library? I think the intention is pretty clear - Go was developed to facilitate service development and concurrent programming, and its standard library attests to it.

### Conclusion

Thank you to the bright minds who have created this amazing language. Coding is now fun again, and I think learning Go has played a major part in it.

Thank you for reading this long post! This is my first attempt at writing a dev blog, so any feedbacks are welcome. If you have your own thoughts that you would like to share, please comment down below!



*Have fun,*

Jacob Kim
