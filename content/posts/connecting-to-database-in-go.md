---
title: "Connecting to Database in Go"
date: 2022-04-17T13:18:45+09:00
draft: true
---

intro

### Start with why

- A database connection is necessary for most web apps.
  
  - There needs to be a way to persist data. In our case, this is the book data.
  
  - We could create a struct and store the data there. This is a bad idea for production use-cases because in-memory storage is volatile. This data will be destroyed when you turn off your database. It's like trying to remember what an image looks like versus actually saving it to your hard drive.
  
  - It's a better idea to use tools that are suited for storing data.

- database/sql package provides support for SQL databases.

- NoSQL databases tend to require third-party packages.

- SQL vs. NoSQL

- Pick your poison depending on your use case.

- For this tutorial, we will use PostgreSQL.

### Houston, do you copy?

- We know why we need a database and have decided to use PostgreSQL. Now we need to actually make a connection.

- Almost all controllers will need to access the database at some point.

- There are two main ways to accomplish a database connection:

##### Using a global variable

```go
// main.go
package main

import (
    "database/sql"
    "fmt"
    "log"

    "example.com/boo"

    _"github.com/lib/pq"
)

const (
    HOST = "localhost"
    PORT = 5432
    USER = "jacob"
    PASSWORD = "password"
    DBNAME = "bookstoreDB"
)

func main() {
    connString := fmt.Sprintf(
        "host=%s port=%d user=%s password=%s dbname=%s",
        HOST, PORT, USER, PASSWORD, DBNAME,
    )

    models.DB, err := sql.Open("postgres", connString)
    if err != nil {
        log.Fatal(err)
    }
    defer models.DB.Close()

    // mux definition and route registration
}
```

- Very simple, good for small projects

- Easy to understand

- Global variable is unsafe and unreliable

- models.go includes a var DB *sql.Db declaration, then you have an InitDB() function to initialize the DB.

- You could just initialize the DB inside main, but this makes the logic unorganized because not all DB-related code is stored in models.go

- Plus, this makes testing harder, because you can’t test main functions.

##### Using dependency injection

- Dependency injection via custom type

- Safer, can scale easier

- Connect to database

- Database is hosted in a separate docker container for simple use.

- Create a docker-compsoe.yml and run the container.

- Connect using sql.Open()

- It is a good idea to store connection data as constants.

### I'll have a grande pumpkin spice latte with non-fat milk

- Querying

- Map GET requests with SELECT.

- Map POST requests with INSERT (usually).

- Map PUT requests with ALTER TABLE.

- Map DELETE requests with DROP.

- We need a struct to store query results.

- It’s a good idea to match the database schema.

- Having this struct is necessary because Go is a statically typed language.

- Store these structs inside a separate models.go file.

- Not required, but it is helpful for organizing code.

- Similarly, move the controllers to controllers.go file.

- A query returns a slice of rows, so you need to iterate through each row and scan it into an instance of your struct.

- Encode the struct object to JSON, and return that.

### Why are we not using an ORM?

- Good question. Go does have two popular options for ORM: ent and GORM.

- I don’t like ORMs because it provides an additional layer of abstraction that separates me from the query.

- Unless I know the source code inside and out, it’s hard to debug things sometimes. This issue tends to happen when queries get very complex.

- I don’t like magic boxes that claim to “just work”. I want to understand how it works.

- Learning SQL is also a good thing as a backend developer. ORM allows you to use the database without knowing SQL, which I think is a bad practice.

- ORMs have their place, but I prefer raw queries.
