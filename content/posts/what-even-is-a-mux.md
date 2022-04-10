---
title: "What Even Is a Mux"
date: 2022-04-10T15:48:48+09:00
draft: true
---

"Engineers are bad at naming things."

This holds true for a lot of things. Weird terms, buzzwords, ten different names for the exact same thing... the list goes on and on. It's a bit embarrasing to admit, but I think one of the most annoying parts of learning Go was how there were jargons and terminologies I had to get used to. This is what you have to go through when you don't know anything in the beginning and decide to dive in head first into a new topic.

What is a mux? What is a handler? What is a middleware? What is the difference between Handle type and Handle function? HandleFunc and HandlerFunc? Is apple an orange? To be or not to be? How do I calculate the distance to the moon when I like to swim and have seven dwarves? What am I even saying?

Fear not, I know what it feels like to get lost in the documentation jungle. This will be part 1 of a series where I try to help beginner Go web developers. In this post, I will explain what a mux is, how it works, and how you can write your own mux. 

### What even is a mux?

Mux. It's not a very common word, that's for sure. Mux is short for multiplexer. What's a multiplexer?

> Multiplexer. A device that enables the simultaneous transmission of several messages or signals over one communications channel. - Dictionary.com

In Go, when we say mux, it usually means a HTTP request multiplexer. It's main job is to match incoming request URLs against a pre-defined set of routes, and do something when there is a match. Simply put, a mux acts as a gateway into your application.

### Creating your custom mux

How might this look like in your code? Let's walk through this step by step.

```go
package main

import (
    "log"
    "net/http"
)

func homeHandler(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("This is the home page."))
}

func aboutHandler(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("This is the about page."))
}

func main() {
    mux := http.NewServeMux()

    mux.HandleFunc("/", homeHandler)
    mux.HandleFunc("/about", aboutHandler)

    log.Fatal(http.ListenAndServe(":8080", mux))
}
```

This is a very simple example backend app. It handles requests to `/` and `/about` endpoints by running the corresponding `homeHandler` and `aboutHandler` handler functions. Let's look at the main function first.

```go
mux := http.NewServeMux()
```

This creates a new instance of `ServeMux`. `ServeMux` is how a mux is defined in the `net/http` code.

```go
mux.HandleFunc("/", homeHandler)
mux.HandleFunc("/about", aboutHandler)
```

A `ServeMux` has a method called `HandleFunc`. This takes in two parameters: the target endpoint, and a handler. `HandleFunc` will register this handler with the target endpoint. Whenever a request hits the endpoint, our `ServeMux` will check if there is a registered handler for that endpoint.

```go
func homeHandler(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("This is the home page."))
}

func aboutHandler(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("This is the about page."))
}
```

These are the handlers. Handlers are what actually does something when an endpoint is reached. They have two parameters that they can work with: the response and the request. It can read any data from the request such as the request header and the body. It can also write any data to the response, whether it be a byte text, a simple HTML, or JSON data.

```go
log.Fatal(http.ListenAndServe(":8080"), mux)
```

This line is what starts the web server. `ListenAndServe` will listen at a designated port, which is 8080 in this case. You can also choose which mux to use, and passing in a `nil` value will make it use the `DefaultServeMux`. Do note that using a default mux isn't recommended, as it is stored as a global variable that other packages can modify.

### How does a mux work?

So we have a high-level understanding of how a mux works. Let's dive a little deeper, because understanding how something works under the hood is a fun thing to do.

```go
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry
	es    []muxEntry
	hosts bool
}

type muxEntry struct {
	h       Handler
	pattern string
}
```

We can ignore `sync.RWMutex` for now, because this is a measure to prevent any issues during concurrent request handling. It is way out of scope for this tutorial.

Like I mentioned above, a mux is defined in Go as a `ServeMux` type, which is a struct that holds a couple of different data.

- `ServeMux.m` holds a map of URL pattern that is paired with `muxEntry`, which is a struct that holds the handler and a URL pattern.

- `ServeMux.es` is an ordered list of `muxEntry` objects used for pattern matching.

- `ServeMux.hosts` is used to match host-based URLs, which includes the full hostname in the URL.

### Conclusion

Hopefully this post was useful as a starting point for beginner Go web developers who want to work on backend web development. Agian, this is just a part 1 of backend web development series, so keep an eye out for the next set of posts! We still haven't covered everything that Go offers us. In the next post, we wil ltake a look at handlers in more detail.

Thank you for reading! You can also view this post on Medium and my personal site.


