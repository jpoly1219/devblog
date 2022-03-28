---
title: "Easy Sorting in Go"
date: 2022-03-28T12:41:50+09:00
draft: false
---

![](/easy-sorting-in-go.png)

Sorting is a popular topic among developers. It is one of the most crucial algorithms in computer science because many important algorithms rely on sorted arrays. For example, a binary search algorithm is commonly regarded as one the fastest way to search for something. However, for it to work, we need to provide a sorted array.

I am not going to delve deeply into sorting algorithms here, because it is out of the scope of this post. However, it is a very interesting topic, so you should definitely look it up. But even if sorting algorithms aren't interesting, you probably came across a situation where you had to sort a list of elements. Instead of implementing your own algorithm from scratch, there is a much easier way to sort things. In this post, we are going to see how we can easily sort things in Go. Enjoy!

### Using the sort package from Go Standard Library

Go usually takes a minimalistic approach to utility functions, so you have to create your own method of doing things in many cases. For example, there is no built-in function to get the minimum or maximum of an array.

Fortunately, Go provides this out of the box because sorting is such a crucial functionality. You can import the `sort` package, and use its functions.

```go
package main

import (
    "fmt"
    "sort"
)

func main() {
    intSlice := []int{5, 3, 9}
    sort.Ints(intSlice)

    f64Slice := []float64{4.8, 10.5, 7.4}
    sort.Float64s(f64Slice)

    stringSlice := []string{"Banana", "Apple", "Cherry"}
    sort.Strings(stringSlice)

    fmt.Println(intSlice, f64Slice, stringSlice)
}
```

```
[3 5 9] [4.8 7.4 10.5] [Apple Banana Cherry]
```

These three basic sorts will get you pretty far, but there is an interesting piece of code in this package.

### sort.Interface

There is an interface that is defined as `sort.Interface`. Here is what the code looks like.

```go
type Interface interface {
    Len() int
    Less(i, j int) bool
    Swap(i, j int)
}
```

Let's take it apart and inspect each function.

- `Len()` just tells you the length of the array.

- `Less()` takes indices `i` and `j` and checks if the i'th element is smaller than the j'th element.

- `Swap()` swaps the order of the i'th and the j'th element.

Some people may have noticed that this is an interface that can be implemented quite easily. What does this mean?

### Implementing your own sorts

Yes, you can create custom sorts. If none of the above three methods suit your needs, just create a type that implements `sort.Interface`, then apply `sort.Sort()` to it. Here is a quick example.

```go
package main

import (
    "fmt"
    "sort"
)

type employee struct {
    name string
    lastname string
    age int
    phoneNumber int
}

type employeeDb []employee

// employeeDb implements sort.Interface
func (edb employeeDb) Len() int {
    return len(edb)
}

func (edb employeeDb) Less(i, j int) bool {
    if edb[i].age == edb[j].age {
        if edb[i].lastname == edb[j].lastname {
            return edb[i].name < edb[j].name
        }
        return edb[i].lastname < edb[j].lastname
    }
    return edb[i].age < edb[j].age
}

func (edb employeeDb) Swap(i, j int) {
    edb[i], edb[j] = edb[j], edb[i]
}

func main() {
    myDb := employeeDb{
        {"Jacob", "Kim", 21, 1234567890},
        {"Chris", "Hemsworth", 38, 9375913934},
        {"Robert", "Downey Jr.", 56, 4459183048},
        {"John", "Doe", 22, 4793721933},
        {"Jane", "Doe", 22, 4792091933},
    }
    sort.Sort(myDb)
    for _, v := range myDb {
        fmt.Println(v)
    }
}
```

```
{Jacob Kim 21 1234567890}
{Jane Doe 22 4792091933}
{John Doe 22 4793721933}
{Chris Hemsworth 38 9375913934}
{Robert Downey Jr. 56 4459183048}
```

Phew, that was a lot of code to swallow. But we can look at the overall flow together.

- We defined a new type `employee`, which is a struct that holds the employee's name, last name, age, and phone number.

- We also defined a new type `employeeDb`, which is an array of `employee`. This type implements `sort.Interface` because it has `Len()`, `Less()`, and `Swap` defined.

- `Len()` just returns the length of `employeeDb`, and `Swap()` just swaps the i'th and the j'th element. `Less()` is a bit long but it's quite simple. 
  
  - We try to determine which value is greater by sorting by age first. If the age is equal, then we try to sort by the last name, then the first name.

- When we `sort.Sort()`, we can see that the employees are sorted by age first.
  
  - Since John Doe and Jane Doe are the same age, they are sorted by their last name. However, they have the same last name as well! So they are sorted by their first name.
  
  - The rest is self-explanatory.

Pretty nifty way to sort things, I must say. Depending on how you play around with `Less()`, you can come up with a totally different way of sorting things.

### Conclusion

This is one of the reasons why I think Go is a cool language. You can just implement an interface to enjoy all of its power. The takeaway here is that the `sort` package provides you with basic sorts, and if they fall short, then implementing `sort.Interface` is an easy way to sort custom types by your own rules.

Hope this post helped! You can read this post on [Medium](https://medium.com/@jpoly1219/easy-sorting-in-go-17525f6f900b) and [Dev.to](https://dev.to/jpoly1219/easy-sorting-in-go-56ae) as well.
