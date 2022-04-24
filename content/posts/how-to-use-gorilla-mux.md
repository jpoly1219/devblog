---
title: "How to Use Gorilla Mux"
date: 2022-04-24T14:20:51+09:00
draft: true
---

Welcome back to my guide, *A Gentle Intro to Golang Web Development*. Last time, we looked at how to write handlers for our web app. Now that we know how muxes and handlers work, it is time to use a more sophisticated tool. While Go does have an amazing `net/http` package, there are certain features that `gorilla/mux` provides that makes our lives easier. Witout further ado, let's get into part 3.

### Why should I use this package to begin with?

If the standard library is so good, then why should we even bother to use `gorilla/mux`? There are a couple of reasons.

- It can extract variables from path.

- It has support for subrouters, which you can use to group similar routes.

- You can match routes via domains, prefixes, methods, and more.

- Along these useful features, you still enjoy a low learning curve thanks to the package implementing the `http.Handler` interface, making it compatible with the standard library.

There are more features that I haven't listed here, but these are the most used features. `gorilla/mux` is a mature package with extensive documentation and a wide userbase, so it will be easier to get help online when you are stuck.

### Importing the package

To use `gorilla/mux`, we need to import it.

```go
package main

import (
    "github.com/gorilla/mux"
)
```

After importing it in your `main.go` file, we need to install it.

```
go mod init example.com/mywebapp

go mod tidy
```

We first initialize our `go.mod` file with the first command. The URL isn't important at the moment. We aren't hosting this in a remote repository. If we were developing a library for other people to use, this this URL would be a path to your remote repo, such as `github.com/username/mypackage`.

Once a `go.mod` file has been created, we use the second line to check and update the dependencies. So `go mod tidy` is usually used to prune unnecessary dependencies, but it is useful for adding missing ones as well.

Now that it is installed, we can start using the package!

### Let's walk through an example with me

Over the course of the guide, we are going to build a library (as in books) management application, where users can search for books and authors. I guess this is similar to Kindle or Bornes & Noble shops, albeit MUCH simpler.

Our web app is going to support these operations:

- Users can get data for all books.

- Users can get data for each book by their ISBN numbers.

- Users can add new books.

- Users can update existing book data.

- Users can delete books.

Of course, some of these operations would be devastating if normal customers could use them freely. Imagine me going to the Kindle ebook store and wiping half of their library. However, since authentication and authorization is out of scope for this guide, we are just going to assume that the users of this web app are managers and admins.

```go
package main

import (
    "fmt"
    "log"
    "net/http"

    "github.com/gorilla/mux"
)

func main() {
    r := mux.NewRouter()

    r.HandleFunc("/", homeHandler)
    
    booksSubR := r.PathPrefix("/books").Subrouter()

    booksSubR.HandleFunc("/all", AllHandler).Methods(http.MethodGet)
    booksSubR.HandleFunc("/{isbn}", IspnHandler).Methods(http.MethodGet)
    booksSubR.HandleFunc("/new", NewHandler).Methods(http.MethodPost)
    booksSubR.HandleFunc("/update", UpdateHandler).Methods(http.MethodPut)
    booksSubR.HandleFunc("/delete/{isbn}", DeleteIspnHandler).Methods(http.MethodDelete)

    log.Fatal(http.ListenAndServe(":8090", r))
}
```

This is a lot of code, but if you already read my previous guides, this example will look familiar.

```go
// using gorilla/mux
r := mux.NewRouter()
r.HandleFunc("/", homeHandler)
```

We first create our mux and register an endpoint. In vanilla Go, we would acomplish by doing this:

```go
// using net/http
mux := http.NewServeMux()
mux.HandleFunc("/", homeHandler)
```

Notice how similar the code is? They basically do the same thing. This is all thanks to `gorilla/mux` implementing the `http.Handler` interface, being compatible with the `net/http` standard library.

This is the part where it gets interesting.

```go
booksSubR := r.PathPrefix("/books").Subrouter()
```

This is where the concept of *matching routes* and *subrouters* come into play. The first method is `PathPrefix()`, which basically matches routes that start with `/books`, such as `/books/all` or `/books/new`. Now, this by itself is usable, but we would like to group these routes together. We do the grouping by calling the `Subrouter()` method. The above code will create a subrouter that matches routes that only start with `/books`. It will not handle any other routes, such as `/home` or `/welcome`.

```go
booksSubR.HandleFunc("/all", booksAllHandler).Methods(http.MethodGet)
booksSubR.HandleFunc("/{isbn}", booksIspnHandler).Methods(http.MethodGet)
```

This is by in large the same thing as a normal `HandleFunc`, but with extra features. 

The `Methods()` method at the back limits the route to only accept certain HTTP request types. In this case, we only accept `GET` requests.

Now, take a look at the second line. `isbn` is what we will call a *URL parameter*. These are variables inside a path, which can later be extracted in our handler functions. So if I send a request to `/books/000-00-00000-00-0`, I will be able to extract `000-00-00000-00-0` as a variable, then use it to do something.

But hold on, if we make a request to `/books/all`, how does the subrouter tell if `all` is an ISBN number? These situations arise when we have conflicting routes like above. Our subrouter will just match the requests in order from top to bottom, so `/all` will be matched first, then the rest will be matched by `/{isbn}`. This isn't a good practice, and you should try to avoid having conflicting routes. If it is necessary, however, we can add a pattern to the URL parameter.

```go
booksSubR.HandleFunc("/{isbn:^[0-9]{3}[-]{1}[0-9]{2}[-]{1}[0-9]{5}[-]{1}[0-9]{2}[-]{1}[0-9]{1}$}", booksIspnHandler).Methods(http.MethodGet)
```

This looks a little crazy, but it's just allows variables that matches the format of ISBN-13 (anything that looks like `000-00-00000-00-0`).

### Conclusion

We learned how to use `gorilla/mux`, and how to use some of its defining features such as pattern matching, URL parameters and subrouters. There are other router packages out there, like `httprouter` and `go-chi`. I personally have never tried using these, but I've heard many good things about them. Feel free to try them out!

Hope you had fun reading this post. In the next one, I will walk through connecting our web app to a database, and executing some queries. The pacing of this guide might seem too slow for some of you, but I am intentionally making this slower so that the guide is easy to follow along. Please understand!

You can also read this on Medium and my personal site.
