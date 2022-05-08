---
title: "Querying the Database in Go"
date: 2022-05-08T11:04:19+09:00
draft: true
---

Welcome back to my tutorial! Last time, we set up an instance of PostgreSQL database, and connected it to our backend. This time, we will actually use the database and learn how to query it.

### Quick intro to SQL databases

Although we have set up the database connection, chances are that you still don't understand how a database works. Is it like file explorer in my computer? How does it store data? What does it mean to *query* the database? I think SQL syntax gets a lot easier if we have a basic understanding of how a database works. 

PostgreSQL is an example of what is called an *SQL database*. An SQL database is a type of database that stores data in the format of a defined *schema*. Simply put, it is a glorified spreadsheet where each data is stored as a row inside a table. An SQL database will typically have several schemas (tables) that define the format of the data.

In our example, we want a table that stores book data. Our struct definition in Go looks like this:

```go
type BookData struct {
    title string
    author string
    isbn string
}
```

Each book has a title, an author, and an ISBN. The table in our database will look something like this:

| Title           | Author       | ISBN          |
| --------------- | ------------ | ------------- |
| Tools of Titans | Tim Ferriss  | 9781328683786 |
| Siddhartha      | Herman Hesse | 9781529024043 |
| Atomic Habits   | James Clear  | 9780735211292 |

The schema and the struct will be redefined in this guide to better suit our needs.

### Simple SQL syntax

Now that we know how a database stores data, it is time to learn how to use the database. Unless you are using a GUI tool to simplify the operations, you will need to use a special language to manipulate the database. Even if you have a GUI tool, it is a good idea to know how to use SQL.

SQL is short for *Structured Query Language*, and is the primary way to manipulate the database. The syntax reads like English, so it is very easy to understand what each line does. The act of requesting and editing data is called a *query*.

Let's start off with learning a couple of the most important commands. Also, yes, it is idiomatic to type SQL commands in all caps in order to distinguish them from the data. For the sake of organization and readability, you may opt to break the query statement into multiple lines.

```sql
CREATE DATABASE myDatabase;
```

This command creates a database. In our case, we have already created the database in the last tutorial, so we can skip this step.

```sql
CREATE TABLE myTable (
    column1 dataType columnConstraints,
    column2 dataType columnConstraints,
    column3 dataType columnConstraints,
    tableContrainsts
);
```

This is how you create a table like the one in the first part of this tutorial.

- You choose the name of the table.

- You must then specify the column names, their data types, and constraints.

Constraints are options that you assign to each column. For example, a `NOT NULL` constraint will require that the column does not have empty values. A `PRIMARY KEY` constraint will designate the column as unique values that can be used to identify each row.

```sql
SELECT column1 FROM myTable WHERE condition;
```

This is how you select certain data from the table. The command is quite self-explanatory: you choose a column or set of columns from table where a condition is met.

```sql
INSERT INTO myTable (column1, column2, ...)
VALUES (value1, value2, ...);
```

This is how you insert data into certain columns. `value1` gets inserted into `column1`, and vice versa.

```sql
UPDATE myTable
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

This is how you update existing values at given columns.

```sql
DELETE myTable WHERE condition;
```

This is how you delete existing values that match the condition.

### Mise en place

Our database currently has nothing inside it. We should prep it before we can do anything in our backend. Let's create our table first.

```sql
CREATE TABLE books (
    isbn VARCHAR(13) PRIMARY KEY,
    title VARCHAR(100) NOT NULL,
    author VARCHAR(50) NOT NULL
);
```

This script creates a table named `books` that has columns `isbn`, `title`, and `author`.

- `VARCHAR(length)` is a type used by strings with maximum length. `VARCHAR(13)` will only accept strings with 13 characters or less. We are going to use ISBN 13 with no hyphens in the middle, so allocating only 13 characters sounds good.

- `PRIMARY KEY` is a constraint that is applied on columns to identify a row uniquely in a table. All items in the `PRIMARY KEY` column must be unique and not be empty.

- `NOT NULL` is a constraint that is used to to ensure no data in the column is empty.

Ok, we created the table. We could seed the database by inserting new rows right now, but we will do this in our backend instead. If you want to add some books now, you can go ahead and use this script:

```sql
INSERT INTO books (isbn, title, author)
VALUES ("9781328683786", "Teen Titans", "Tim Ferriss");

INSERT INTO books (isbn, title, author)
VALUES ("9781529024043", "Siddhartha", "Herman Hesse");

INSERT INTO books (isbn, title, author)
VALUES ("9780735211292", "Atomic Habits", "James Clear");
```

Whoops, we made a typo in our first book. The title should be **Tools of Titans**, not **Teen Titans**! To fix this, we can use this query statement:

```sql
UPDATE books
SET title = "Tools of Titans"
WHERE isbn = "9781328683786";
```

Pretty straighforward, right?

### Executing queries in Go

Good job following the guide so far! Now we will learn how to execute these queries from our backend. Let's go back to out code.

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

We should define the handlers that fire when a corresponding endpoint is reached. To make things less cluttered, let's create a new file named `controllers.go` and write our controllers there. In case you are wondering, I am using the term *controllers* and *handlers* interchangably.

Before we create `controllers.go`, we need to create a defined model that reflects the schema of the database. These models should go inside `models.go`.

```go
// models.go
package main

type Book struct {
    Isbn string `json:"isbn"`
    Title string `json:"title"`
    Author string `json:"author"`
}
```

Now that we have `Book` defined, we can go ahead and use that in our `controllers.go` file.

```go
// controllers.go
package main

import (
    "database/sql"
    "net/http"
)

func AllHandler(w http.ResponseWriter, r *http.Request) {
    var books = make([]Book, 0)

    results, err := DB.Query("SELECT * FROM books;")
    if err != nil {
        log.Println("failed to execute query", err)
        w.WriteHeader(500)
        return
    }

    for results.Next() {
        var b Book

        err = results.Scan(&b.isbn, &b.title, &b.author)
        if err != nil {
            log.Println("failed to scan", err)
            w.WriteHeader(500)
            return
        }

        books = append(books, b)
    }

    json.NewEncoder(w).Encode(books)
}

// other handler definitions go here
```

This is an example of how a controller function can query the database, parse the results as JSON, then send it back to the user.

```go
var books = make([]Book, 0)
```

We create a slice of `Book` objects. This is used to store the query results.

```go
results, err := DB.Query("SELECT * FROM books;")
if err != nil {
    log.Println("failed to execute query", err)
    w.WriteHeader(500)
    return
}
```

We haven't seen this code before. This is the snippet that executes the query.

- We created `DB` in the `main` function. We use the `Query` method on `DB` to execute a query.

- The query statement is a string: `SELECT * FROM books;`. The asterisk denotes that we want to select all records from the table `books`.

- The method returns `results`, which is of type `sql.Rows`. This stores the result of a query, row by row. Getting this alone isn't enough; we need to scan the contents into another variable.

- If there is an error, we respond back with an HTTP error code of 500, which denotes that there was an issue on the server-side.

```go
for results.Next() {
    var b Book

    err = results.Scan(&b.isbn, &b.title, &b.author)
    if err != nil {
        log.Println("failed to scan", err)
        w.WriteHeader(500)
        return
    }

    books = append(books, b)
}
```

This is where we save the query results.

- `results.Next` iterates through the rows of `results`. Because we queried for all records, there are many rows that we need to iterate through.

- `results.Scan` scans the row data to the target destination. Here, we are scanning to an instance of `Book` called `b`.

- Once the scanning is complete, we append `b` to the `books` slice.

- If there is an error, we respond back with an HTTP error code of 500, which denotes that there was an issue on the server-side.

```go
json.NewEncoder(w).Encode(books)
```

This is simple. It encodes the `books` slice into JSON, then sends it back to the user.

### Handling POST requests

We need to handle requests to add new books to our database. The basics are the same as the previous example, but the querying method will be slightly different.

```go
// controllers.go
package main

import (
    "database/sql"
    "net/http"
)

func NewHandler(w http.ResponseWriter, r *http.Request) {
    var b Book

    err := json.NewDecoder(r.Body).Decode(&b)
    if err != nil {
        log.Println("error while decoding r.Body", err)
        w.WriteHeader(400)
        return
    }
    
    queryStmt := `
        INSERT INTO books (isbn, title, author)
        VALUES ($1, $2, $3)
        RETURNING isbn;
    `
    var isbn string
    err := DB.QueryRow(queryStmt, &b.isbn, &b.title, &b.author).Scan(&isbn)
    if err != nil {
        log.Println("failed to execute query", err)
        w.WriteHeader(500)
        return
    }

    log.Println("%s has been added to the database", isbn)
}
```

The code looks similar to our first example, but it is slightly different.

```go
var b Book

err := json.NewDecoder(r.Body).Decode(&b)
if err != nil {
    log.Println("error while decoding r.Body", err)
    w.WriteHeader(400)
    return
}
```

- Because the user will send a JSON object as a request body, we need to marshal it in a Go-friendly format.

- We create an instance of `Book` called `b`, and we decode the request body to it.

- If an error occurs, this means that the request body is invalid, so we return an error code of 400.

```go
queryStmt := `
    INSERT INTO books (isbn, title, author)
    VALUES ($1, $2, $3)
    RETURNING isbn;
`
var isbn string
err := DB.QueryRow(queryStmt, &b.isbn, &b.title, &b.author).Scan(&isbn)
if err != nil {
    log.Println("failed to execute query", err)
    w.WriteHeader(500)
    return
}

log.Println("%s has been added to the database", isbn)
```

This is how we handle `INSERT` statements, or any statements that require us to dynamically create new query statements such as `UPDATE`.

- `queryStmt` is the query statement. Focus on the second line - it has `$1, $2, $3`. These are placeholders for data we will insert later. Because one person's `INSERT` statement is going to be different from someone else's, we can't just have a static query statement. How are we to know what book the users want to add before they even tell us?

- The `RETURNING isbn;` line at the end will return the `isbn` of the book added. This is a useful data to keep, and we can use it to do whatever we please.

- Based on the query statement, we know for sure that there is only going to be one row returned after the query. This is why we use `sql.QueryRow` instead of `sql.Query`.

- In `DB.QueryRow`, we pass in our query statement, and the data to replace `$1, $2, $3` with. Order matters, so in our example, `&b.isbn` will replace `$1`, and vice versa. `&b.isbn`, `&b.title`, and `&b.author` are the fields of `b` that we have decoded previously.

- If there is an error, we respond back with an HTTP error code of 500, which denotes that there was an issue on the server-side.

You will be able to figure out how to write other controller functions from here. Best of luck! Check out these resources to help you:

- [sql package - database/sql - pkg.go.dev](https://pkg.go.dev/database/sql)

- [PostgreSQL Tutorial](https://www.postgresqltutorial.com)

### Conclusion

Congratulations on completing this part of the tutorial. We now have a very basic REST API up and running. Run the code, fire up your favorite client (mine is cURL and Postman), and start testing! Feel free to add more controllers and expand. There are more advanced topics in building APIs, but for learning purposes, this will be enough. I will get to more advanced topics in the near future, so stay tuned!

You can also read this post on Medium and my personal site.
