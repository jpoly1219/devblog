---
title: "How Slices Work in Go"
date: 2022-03-21T12:18:53+09:00
draft: false
---

Go provides a way for us to group similar data using slices. I guess it's a rather unfamiliar term. At least I haven't seen the term slice being used in any other languages. Being able to use slices well is important, but understanding how it works under the hood is important as well. I think understanding the inner workings is half the fun. In this post, I will explain how slices work in Go.

### But first, let's look at arrays.

The reason why we are starting with arrays is that slices are implemented using arrays. Arrays are basically containers with fixed sizes.

```go
myArray := [3]int{0, 1, 2}
```

You specify the size of the array inside the square brackets, decide on the data type of the elements, and write the elements inside the curly braces. You could also let the compiler find out the length of the array by doing this:

```go
myArray := [...]string{"apple", "banana"}
```

If you do not specify the length, a slice will be created instead. You cannot change the size of an array once it has been created.

So already, you can see that arrays are more primitive and not as feature-rich as, say, a Python list would. I come from a Python background, and like a spoiled newbie dev I am, wanted to find something more useful. Immediately I was looking for something more powerful.

### Behold, the creation of slices.

Slices are much more powerful than arrays, because of their dynamic nature. Simply put, a slice is like a rubber band. It will expand as much as you need as you go. A rubber band won't be very useful if it was hard and can't stretch.

It's similar to how C++ developers can use both arrays and vectors but will opt to use vectors for its dynamic-ness.

Here are some ways that you can initialize a slice.

```go
// method 1: use make() and specify type, length, and capacity.
mySlice := make([]int, 4, 4)
// assign values to each index.
mySlice[0] = 0
mySlice[1] = 1
mySlice[2] = 2
mySlice[3] = 3

// method 2: declare a struct literal.
mySlice := []int{0, 1, 2, 3}

// method 3: create an empty slice and append to it
mySlice := []int{}
mySlice = append(mySlice, 0, 1, 2, 3)
```

Slices are nice because you can change their size after you create them. This is especially handy when you need to store an unknown amount of data. Slices also support slicing (duh), which lets you take slices of a given slice.

```go
mySlice := []int{0, 1, 2, 3}
piece := mySlice[1:3]
```

```
[1 2]
```

So already, we see a major improvement in usability. But when I first saw this, I couldn't help but wonder how this worked.

### How does this work? What is this sorcery?

So under the surface, a slice is a header that contains a pointer to an underlying array. If you look at the `reflect` package of go, you can see the definition of `SliceHeader`:

```go
type SliceHeader struct {
    Data uintptr
    Len int
    Cap int
}
```

So because it is an array at heart, a slice cannot outgrow its capacity. When we say "expanding the slice", we don't actually mean to add more capacity to what we already have. Instead, it's this behind-the-scene logic that allows the slice to "grow".

- It will check that the current length is equal to the capacity.

- If appending over-capacity, a new slice with double the original slice's capacity will be created.

- The original slice will be copied over to the new slice.

- The new element will be appended at the end.

- The resulting slice will be returned.

![](/grow-slices-go.png)

Slicing is easy as well:

- It will point to a new location in the *same* underlying array.

- Length and capacity will be adjusted.

![](/slicing-slices-go.png)

So we now understand how slices work under the hood. Cool, isn't it?

### Conclusion

You can think of slices as a wrapper around arrays to give them superpowers. In most cases, you should try to use slices instead of arrays. However, when you know what you are doing, arrays can be a fine choice. If you need something that has to be passed by value, or be hashed or serialized, you should use arrays over slices. However, sticking with slices is the way to go in most cases.

Thanks for reading! You can read this post on [Medium](https://medium.com/@jpoly1219/how-slices-work-in-go-fbc772514001) and [Dev.to](https://dev.to/jpoly1219/how-slices-work-in-go-47nc).
