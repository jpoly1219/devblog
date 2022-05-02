---
title: "Connecting to Database in Go"
date: 2022-04-17T13:18:45+09:00
draft: false
---

Welcome back to *A Gentle Intro to Golang Web Development*. Last time, we learned how to use `gorilla/mux`. This time, we will learn how to set up a database and connect to it. Let's get started!

### Start with why

It's important to understand why we need a database. A database connection is necessary for most web apps because there needs to be a way to persist data reliably. 

We could create a struct and store the data there. Something like this:

```go
type BookData struct {
    title string
    author string
    isbn string
}


books := []BookData{
    {"Tools of Titans", "Tim Ferriss", "978-13-28683-78-6"},
    {"Siddhartha", "Herman Hesse", "978-1529024043"},
}
```

Sure, this might store the book data, but in our RAM. This is a bad idea because in-memory storage is volatile. The data stored in RAM will be destroyed when you turn off your database.

Imagine going on a trip to France. You see the Eiffel Tower, eat haute-cuisine from the best restaurants, and make new friends. Storing it to RAM would be trying to remember all these cherished memories in your mind. Using a database is like taking pictures and saving them to your computer.

For our gamer friends (like me!), imagine your account data resetting every time there is server maintenance. Yikes.

It's a good idea to use tools suited for storing data. `database/sql` package provides support for SQL databases. If you want to use NoSQL databases, you will most likely require third-party packages. For this tutorial, we will use PostgreSQL.

### Create the database

The easy way to create a database is to use a Docker image. Docker is a tool that lets us run services inside a container. Containerizing services allows them to run in an isolated environment, making it easy to run, stop and delete.

First, let's install Docker. I highly recommend you follow [this tutorial](https://docs.docker.com/get-started/) to get your hands dirty with simple Docker commands. This tutorial will get Docker installed, and help you understand what Docker is and how it works.

Once you're done with the tutorial, we can start working on our containers. Create a file named `docker-compose.yml` and edit it like this:

```docker
version: "3.7"

services:
  database:
    container_name: database
    image: postgres
    ports:
      - "5432:5432"
    volumes:
      - ./database:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=bookstoreDB
      - POSTGRES_USER=jacob
      - POSTGRES_PASSWORD=password
```

The file above does a few things:

- It pulls the `postgres` image from the Docker Hub.

- The container opens port 5432 for us to connect.

- It sets the environment variables to use when connecting to the database.

- POSTGRES_USER and POSTGRES_PASSWORD  can be anything. For this tutorial, I'm using `jacob` and `password`.

- Thanks to volume, data persists inside the container at `/var/lib/postgresql/data`.

We can run this by using the following command:

```
docker-compose up -d
```

We can stop this by using the following command:

```
docker-compose down
```

Once the container is running, we can connect to the database by using a tool called pgAdmin. Go ahead and install pgAdmin for your OS [here](https://www.pgadmin.org/download/). Once installed, open the app. To the left, right-click on `Servers`, and go to `Create > Server`.

- In the General tab, name this whatever you want. Go to the Connection tab.

- The hostname/address is `localhost`.

- Port is `5432`.

- The maintenance database is `bookstoreDB`.

- Username and password are what you set in the `docker-compose.yml` file.

Once you save, pgAdmin will create a new connection. Once we drop down to `bookstoreDB`, we can click on `Tools > Query Tool` at the top toolbar. This opens a window where we can type and execute SQL queries.

Ok, the setup process was long, but we don't have to touch it often afterwards. Let's move on to the Go code.

### Houston, do you copy?

We know why we need a database and have created an instance of PostgreSQL. Now we need to make a connection from our Go program.

```go
// main.go
package main

import (
    "database/sql"
    "fmt"
    "log"

    "github.com/gorilla/mux"
    _"github.com/lib/pq"
)

var DB *sql.DB

const (
    HOST = "localhost"
    PORT = 5432
    USER = "jacob"
    PASSWORD = "password"
    DBNAME = "bookstoreDB"
)

func main() {
    connString := fmt.Sprintf(
        "host=%s port=%d user=%s password=%s dbname=%s sslmode=disable",
        HOST, PORT, USER, PASSWORD, DBNAME,
    )

    DB, err := sql.Open("postgres", connString)
    if err != nil {
        log.Fatal(err)
    }
    defer DB.Close()

    // mux definition and route registration (from last tutorial)
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

This is a very simple, easy-to-understand way to create a connection.

```go
import (
    "database/sql"
    "fmt"
    "log"

    "github.com/gorilla/mux"
    _"github.com/lib/pq"   
)
```

That last import statement looks suspicious. Why is there an underscore at the beginning? Also, why even download this package when Go already provides the `database/sql` package? These are all great questions!

- `database/sql` is designed to support many SQL and SQL-like databases. It does not include drivers for all the databases out there. The package developers thought that it is more efficient to separate the drivers from the `database/sql` package. Think of it as a modular travel adapter where the power brick stays the same, but the prongs are swappable.

- `github.com/lib/pq` is the driver for PostgreSQL. This package is a little different in that there is nowhere in the code where we use this driver. Therefore, we put an underscore at the front to denote that the driver is being used but not actively called by the user.

Make sure to run `go mod tidy` to update the dependencies list.

```go
var DB *sql.DB
```

- We store connection data as a global variable using `var DB *sql.DB`.

- The database is represented as a pointer to the struct `sql.DB`, because DB itself is a large struct. It is more efficient to take the pointer of it. 

If you don't know what a pointer is, it's a variable that stores the memory address of an object. 

- An object such as a large struct with multiple attributes takes up a lot of space. Passing it around is cumbersome, because the program needs to copy a lot of data every time.

- If we use a pointer instead, we are just passing around a reference to the object, which is a lot lighter.

- We tend to create shortcuts to certain folders when we organize our files, right? We don't have to copy the file to our desktop. We just have to create a shortcut, which saves a lot of precious gigabytes.

I will write a guide on pointers some day, so that more people can get help.

```go
const (
    HOST = "localhost"
    PORT = 5432
    USER = "jacob"
    PASSWORD = "password"
    DBNAME = "bookstoreDB"
)

func main() {
    connString := fmt.Sprintf(
        "host=%s port=%d user=%s password=%s dbname=%s sslmode=disable",
        HOST, PORT, USER, PASSWORD, DBNAME,
    )

    DB, err := sql.Open("postgres", connString)
    if err != nil {
        log.Fatal(err)
    }
    defer DB.Close()
}
```

- To connect to our database, we need to define the host, port, username, password, dbname, and SSL mode. These won't change, so it's a good idea to declare them as constants.

- The values follow the environment variables we defined in our `docker-compse.yml` file. 

- `connString` is a string that holds all these variables, passed to `sql.Open()` as an option.

- `sql.Open()` connects our program to the database. Its return value is stored in our `DB` variable.

- We need to close our connection when the app is not running, so we use the `defer` keyword to close the connection right before the app stops.

Using global variables works for simple apps like this, but it is not recommended in production because of how unsafe global variables are to mutation. Simply put, we have no control over what happens to the global variable. As you move on, dependency injection will be used more often. However, this concept is out of the scope of today's tutorial.

### Conclusion

Thank you for reading! I hoped to cover querying today, but that would make the tutorial too long. Plus, introducing new tools such as Docker and PostgreSQL is already lot to swallow. The next tutorial will be all about querying the database, so stay tuned!

You can also read this post on [Medium](https://medium.com/@jpoly1219/connecting-to-database-in-go-f98c3a7a06a8) and [Dev.to](https://dev.to/jpoly1219/connecting-to-database-in-go-4m95)
