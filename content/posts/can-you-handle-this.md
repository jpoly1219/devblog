---
title: "Can You Handle This"
date: 2022-04-17T15:26:15+09:00
draft: false
---

Welcome to part 2 of my ongoing series, "A Gentle Intro to Golang Web Development." I hope you enjoy it!

Last time, we took a look at muxes. We know that they act as routers that route requests based on matching endpoint patterns. Now it's time to look at how the requests get handled.

### Handlers, what are they?

Handlers are objects that "handle" HTTP requests. All handlers implement the `Handler` interface shown below:

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}

type ResponseWriter interface {
	Header() Header
	Write([]byte) (int, error)
	WriteHeader(statusCode int)
}

type Request struct {
	Method string
	URL *url.URL
	Proto      string
	ProtoMajor int
	ProtoMinor int
	Header Header
	Body io.ReadCloser
	GetBody func() (io.ReadCloser, error)
	ContentLength int64
	TransferEncoding []string
	Close bool
	Host string
	Form url.Values
	PostForm url.Values
	MultipartForm *multipart.Form
	Trailer Header
	RemoteAddr string
	RequestURI string
	TLS *tls.ConnectionState
	Cancel <-chan struct{}
	Response *Response
}
```

So for an object to become a handler, it needs a `ServeHTTP()` function. `Responsewriter` is an interface built on `io.writer` that lets you write HTTP responses. `*Request` is a pointer to the `Request` struct from which you can read HTTP request data by accessing `Request.Body`.

If you are familiar with the concept of MVC, handlers are conceptually similar to controllers.

### Creating your custom Handler

It's easier to understand the concept when I show you how it works in an example code.

```go
package main

import (
	"log"
	"net/http"
)

type homeHandler struct {
}

func (hh homeHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Welcome!"))
}

func main() {
	mux := http.NewServeMux()

	hh := homeHandler{}

	mux.Handle("/home", hh)

	log.Fatal(http.ListenAndServe(":8080", mux))
}	
```

Let's take this apart and tackle it bit by bit.

```go
type homeHandler struct {
}

func (hh homeHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Welcome!"))
}
```

- `homeHandler` is a struct that implements the `Handler` interface, because it has a struct method `ServeHTTP()` defined.

- Our `ServeHTTP()` method just writes a string `"Welcome!"` to the HTTP response.

```go
func main() {
	mux := http.NewServeMux()

	hh := homeHandler{}

	mux.Handle("/home", hh)

	log.Fatal(http.ListenAndServe(":8080", mux))
}	
```

- We define our custom mux in the main function by calling the `http.NewServeMux()`. 

- Then we create an instance of our `homeHandler` and name it `hh`. 

- We now register a route to our mux by calling `mux.Handle()`. This registration associates the route `/home` with the handler `hh`, so that whatever request that is sent to `/home` will be handled by `hh`. 

- Finally, the last line hosts the web server at port 8080, using our custom mux as a handler to handle incoming requests (yes, a mux is a special type of `Handler` that returns a `Handler` instead of writing to `http.ResponseWriter`).

### A quicker way to create Handlers

The method describe above is good and all, but it is somewhat verbose. Making a custom struct and defining `ServeHTTP()` for it, then creating an instance of the handler to handle one route seems like overkill.

Fortunately, there is an easier way to do it. We can just use functions as handlers instead of creating structs with `ServeHTTP()` methods.

```go
package main

import (
	"log"
	"net/http"
)

func homeHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Welcome!"))
}

func main() {
	mux := http.NewServeMux()

	hh := http.HandlerFunc(homeHandler)

	mux.Handle("/home", hh)

	log.Fatal(http.ListenAndServe(":8080", mux))
}
```

Keen viewers may have picked up something odd. `homeHandler()` doesn't have `ServeHTTP()` defined anywhere. So this must not be a handler, right?

Yesn't.

You are right, `homeHandler` isn't technically a handler. However, it is something else; a `HandlerFunc`.

```go
type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```

The definition of the `HandlerFunc` type might be a bit weird at first. It just means that any function that accepts `ResponseWriter` and `*Request` is a `HandlerFunc`.

A `HandlerFunc` has its own `ServeHTTP()` method, so it implements the `Handler` interface. The method calls the function that is defined by the `HandlerFunc`. In our example, we are just writing `"Welcome!"` to the HTTP response.

```go
func homeHandler(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Welcome!"))
}

func main() {
	mux := http.NewServeMux()

	hh := http.HandlerFunc(homeHandler)

	mux.Handle("/home", hh)

	log.Fatal(http.ListenAndServe(":8080", mux))
}
```

Going back to our example, look at where we define `hh`. Because `homeHandler` isn't a handler on its own, we need to cast it to a `HandlerFunc` type, which does implement the `Handler` interface. We can then pass `hh` to `mux.Handle()`.

There's an even faster way to do this:

```go
func main() {
	mux := http.NewServeMux()

	mux.HandleFunc("/home", homeHandler)

	log.Fatal(http.ListenAndServe(":8080", mux))
}
```

`HandleFunc` is `mux.Handle` for functions. It takes an `HandlerFunc` as an input, so you don't even have to cast your function to the `HandlerFunc` type.

### Conclusion

I hope you enjoyed reading this post! This series aims to help new Go web developers familiarize themselves with web development concepts. I want to share my knowledge by writing easy-to-understand tutorials, while also explaining the "why" along with the "how".

Storytime: I started my Go journey by just ripping off tutorials. However, this made me uncomfortable because I still didn't know what the heck I was doing. I got lost immediately if a tutorial code was written in a different style using a different approach. That's when I decided to look into the code and understand why things are the way they are. And I thought to myself, "There are probably other people in similar shoes as I am. I should write beginner-focused tutorials to help spread the knowledge." My posts may be but several grains of sand in a beach of tutorials. But I believe that after reading my blog posts, someone out there might have an "aha" moment.

Be on the lookout for the next article coming next week. See you then!

This post is also available on Medium and my personal site.
