---
title: "Reading and Writing Different Files in Go"
date: 2022-08-07T09:05:39+09:00
draft: false
---

Reading and writing files is an important feature of your program. Not all data is stored in the same memory space as your program, and sometimes you will need to share data with other programs or view it later with a different program. Storing your data in a file is a good way to achieve these goals. Today, we will look at how you can read from and write to commonly used file types.

### io.Reader, io.Writer

Have you ever wondered how Go can read and write so many different things? It's all thanks to these powerful interfaces. `io.Reader` describes anything that `Read`'s, and `io.Writer` describes anything that `Write`'s. Because this behavior can be replicated easily, anything that implements the `io.Reader` and `io.Writer` interfaces can be used for I/O operations. This means that you can plug and play different inputs and outputs. You could read from a CSV file and output it to JSON, or you could read from a `stdin` and write to CSV.

### CSV

A CSV file consists of rows of data, where each value is separated by a comma (hence the name comma separated values). CSV isn't the fastest format to read and write out there, but it is very versatile and can be understood by many other tools.

#### Reading

```go
package main

import (
    "encoding/csv"
    "fmt"
    "os"
)

func main() {
    f, err := os.Open("see-es-vee.csv")
    if err != nil {
        fmt.Println(err)
    }
    defer f.Close()

    r := csv.NewReader(f)

    records, err := r.ReadAll()
    if err != nil {
        fmt.Println(err)
    }

    fmt.Println(records)
}
```

```
[[id fruit color taste] [0 apple red sweet] [1 banana yellow sweet] [2 lemon yellow sour] [3 grapefruit red sour]]
```

Here is a very simple way to read data from a CSV file.

- We first open our file `see-es-vee.csv` and save that instance as `f`. 

- We want to close `f` once we are done. This is a good way to save memory. Although this is a short program and closing isn't all that necessary, it is good to get in the habit of doing so.

- `f` is of type `*os.File`, which implements the `io.Reader` interface. Therefore, we can pass `f` to `csv.NewReader`. 

- This returns a `csv.Reader` object `r`. `csv.Reader` is a type of `io.Reader` that specializes in reading CSV files. 

- Each line of a CSV file is called a *record*. So we can think of a CSV file as a *slice of records*. `r.ReadAll` returns this slice of records.

- If we print `records`, we can see a 2D slice of rows.

This is great, but what if we want to apply some operations to each record as we read? Fortunately, we can take a more granular approach.

```go
func main() {
    f, err := os.Open("see-es-vee.csv")
    if err != nil {
        fmt.Println(err)
    }
    defer f.Close()

    r := csv.NewReader(f)

    for {
        record, err := r.Read()
        if err != io.EOF {
            break
        }
        if err != nil {
            fmt.Println(err)
        }

        fmt.Println(record)
    }
}
```

Looks similar to the above, right? The part until creating `r` is the same. Let's see what happens next.

- We enter an infinite for loop, because we want to read the file line by line, and Go has no idea how long the file is.

- We read each record using `r.Read`.

- How do we know if we reached the end of the file? `r.Read` returns a special error called `io.EOF`. EOF stands for *end of file*. We catch this error first and tell our program to break out of the for loop once we reach the end.

- For any other errors, you can just handle them the way you would normally do.

- After we handle errors, we can do whatever we want with the record that has been extracted. Some ideas that I can think of are capitalization, rounding, comparing it to an arbitrary value, etc.

Now let's look at how to write to a CSV file.

```go
package main

import (
    "encoding/csv"
    "os"
)

func main() {
    f, err := os.Create("output.csv")
    if err != nil {
        panic(err)
    }
    defer f.Close()

    w := csv.NewWriter(f)

    records := [][]string{
        {"id", "fruit", "color", "taste"},
        {"0", "apple", "red", "sweet"},
        {"1", "banana", "yellow", "sweet"},
        {"2", "lemon", "yellow", "sour"},
        {"3", "grapefruit", "red", "sour"},
    }

    w.WriteAll(records)
}
```

```
id,fruit,color,taste
0,apple,red,sweet
1,banana,yellow,sweet
2,lemon,yellow,sour
3,grapefruit,red,sour
```

Very simple and straightforward.

- We first need to create a file which we can dump our data into. We are going to call this `output.csv`. We just call `os.Create`, and save an instance of it as `f`.

- Remember how we created a `csv.Reader` before? Here we are creating a `csv.Writer` object `w`. `csv.NewWriter` accepts the `io.Writer` interface, which `f` implements.

- We prepare our data as a 2D slice of strings. We will call this `records`.

- Finally, we just use `w.WriteAll` to write `records` to `output.csv`.

### JSON

Because Go is used extensively in web services, it has robust support for JSON. I wrote an entire post about reading and writing JSON files, so check that out for a more in-depth guide!

#### Reading

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
)

type fruit struct {
    Id    int    `json:"id"`
    Fruit string `json:"fruit"`
    Color string `json:"color"`
    Taste string `json:"taste"`
}

func main() {
    f, err := os.Open("jay-son.json")
    if err != nil {
        fmt.Println(err)
    }
    defer f.Close()

    dec := json.NewDecoder(f)

    // read opening bracket
    _, err = dec.Token()
    if err != nil {
        fmt.Println(err)
    }

    for dec.More() {
        var fr fruit
        err := dec.Decode(&fr)
        if err != nil {
            fmt.Println(err)
        }

        fmt.Println(fr)
    }

    // read closing bracket
    _, err = dec.Token()
    if err != nil {
        fmt.Println(err)
    }
}
```

```
{0 apple red sweet}
{1 banana yellow sweet}
{2 lemon yellow sour}
{3 grapefruit red sour}
```

```json
// jay-son.json
[
    {"id": 0, "fruit": "apple", "color": "red", "taste": "sweet"},
    {"id": 1, "fruit": "banana", "color": "yellow", "taste": "sweet"},
    {"id": 2, "fruit": "lemon", "color": "yellow", "taste": "sour"},
    {"id": 3, "fruit": "grapefruit", "color": "red", "taste": "sour"}
]
```

Here's a simple way to read JSON files. Normally, JSONs come in streams. That is, they come in a list of objects. Go handles streams via a decoder.

Let's take a look at this first:

```go
type fruit struct {
    Id    int    `json:"id"`
    Fruit string `json:"fruit"`
    Color string `json:"color"`
    Taste string `json:"taste"`
}
```

This struct acts as a model. Because JSON can come in various shapes and sizes, you ideally want to have a model that mirrors the JSON's structure. The struct fields need to be public and should have a tag to the right which denotes what key it refers to. If you don't know what the JSON is going to look like in advance, Go will just use a `map[string]interface{}`.

- First, we open the file and save its instance to `f`. Don't forget to defer a call to close `f` later!

- We create our decoder object `dec` using `json.NewDecoder`. Just like `csv.NewReader`, it accepts an `io.Reader`. You can start to see the power of interfaces - the details of reading are abstracted, and the workflow for reading is consistent across so many different file types.

- Once `dec` is created, we can read our JSON. Just one problem, though. We need to make sure that we catch the opening and closing brackets using `dec.Token`. Not doing so is like trying to eat a subway sandwich with the wrapper still on. Bleh.

- Just like how we read the CSV file line by line, we read the JSON stream object by object. We loop over `dec.More`, which returns `true` as long as there are objects still left to read.

- We create an instance of `fruit` to store our object's data. Use `dec.Decode` to dump the object data to `f`. You can do what you want with this now.

- After you are done reading, make sure to catch the closing bracket.

#### Writing

Writing to a JSON file is simple as well. We call this *encoding*.

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
)

type Fruit struct {
    Id    int    `json:"id"`
    Fruit string `json:"fruit"`
    Color string `json:"color"`
    Taste string `json:"taste"`
}

func main() {
    f, err := os.Create("output.json")
    if err != nil {
        panic(err)
    }
    defer f.Close()

    enc := json.NewEncoder(f)

    apple := Fruit{Id: 0, Fruit: "apple", Color: "red", Taste: "sweet"}
    banana := Fruit{Id: 1, Fruit: "banana", Color: "yellow", Taste: "sweet"}
    lemon := Fruit{Id: 2, Fruit: "lemon", Color: "yellow", Taste: "sour"}
    grapefruit := Fruit{Id: 3, Fruit: "grapefruit", Color: "red", Taste: "sour"}

    fruits := []Fruit{apple, banana, lemon, grapefruit}

    err = enc.Encode(fruits)
    if err != nil {
        fmt.Println(err)
    }
}
```

```
[{"id":0,"fruit":"apple","color":"red","taste":"sweet"},{"id":1,"fruit":"banana","color":"yellow","taste":"sweet"},{"id":2,"fruit":"lemon","color":"yellow","taste":"sour"},{"id":3,"fruit":"grapefruit","color":"red","taste":"sour"}]
```

This one's also very straightforward.

- We create a file named `output.json` to dump our data into.

- We create a new encoder `enc` by using `json.NewEncoder`.

- We prep our data, which is a slice of `Fruit` objects.

- Finally, we encode this slice by using `enc.Encode`.

### Excel

Go does not support reading and writing Excel files by default. However, there is a popular library called `qax-os/excelize` that helps you do this. If you take apart the source code, you can see that the package uses `*os.File` extensively, which is also an `io.Reader` and `io.Writer`. I think this shows the beauty of the `io.Reader` and `io.Writer` interfaces, because with a bit of tweaking, you can make a custom reader and writer that implements those interfaces, allowing you to support more file types.

### Conclusion

Hopefully, this post served as a quick intro to reading and writing files in Go, and how powerful the `io.Reader` and `io.Writer` interfaces are. I think this is one of Go's beauty - interfaces allow for a very flexible and reusable code. There are certainly more files that we haven't covered in this post, but the general gist is the same: open the file, create a reader or a writer, and read/write from it. Thank you!

You can also read this post on [Medium](https://medium.com/@jpoly1219/reading-and-writing-different-files-in-go-9c119edec51a) and [Dev.to](https://dev.to/jpoly1219/reading-and-writing-different-files-in-go-4m3i).
