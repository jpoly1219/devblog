---
title: "Queues in Go"
date: 2022-06-05T15:03:43+09:00
draft: true
---

Welcome back to *Introduction to Data Structures in Go*! In this post, we will be looking at queues. We hear the term "queue" thrown around a lot in the real world. *The queue for the movie tickets is long.* *Man, it's taking forever to queue up for this League match.* Queues  are a familiar concept for most of us.

### Queues as data structures

Queues are a special list, just like stacks. Unlike stacks, however, queues are known for being FIFO data structures. This means that whatever item that goes in first comes out first. You can imagine a line for the bus, where the first person in the line will get on the bus first, while the rest have to wait until their turn. A queue is simply a list where you can only insert at the tail (back) and pop at the head (front).

Popular operations that can be done on queues include:

- **Enqueing**, aka **Pushing** or **Inserting**

- **Dequeing**, aka **Popping** or **Deleting**

- Front

- IsEmpty or IsFull

### Write in Go

Let's start by defining the type.

```go
type Queue struct {
    items []int
}
```

Simple, all we need to do is to define a slice of a certain data type.

Now for the two most important methods.

```go
func (q *Queue) Enqueue(data int) {
    q.items = append(q.items, data)
}

func (q *Queue) Dequeue() {
    q.items = q.items[1:]
}
```

These are also pretty simple. By using `append`, we are inserting new elements to the tail end of `q.items`. When dequeueing, we are just taking a slice of `q.items` from the 1st element to the last. Remember that we count from 0 in programming, so the 0th element is the first one in the list.

Here are some useful helper methods that will make our lives easier.

```go
func (q *Queue) Front() (int, error) {
    if len(q.items) == 0 {
        return 0, fmt.Errorf("queue is empty")
    }

    return q.items[0], nil
}

func (q *Queue) IsEmpty() bool {
    if len(q.items) == 0 {
        return true
    } else {
        return false
    }
}

func (q *Queue) Print() {
    for _, item := range q.items {
        fmt.Print(item, " ")
    }
    fmt.Println()
}
```

`Front` will return an error if the queue is empty, or else it will return the 0th element in the list.

`IsEmpty` checks whether the length of our queue is 0, and returns `true` if so.

`Print` iterates through our items and prints each one.

Let's try using it!

```go
func main() {
    myQueue := Queue{[]int{}}
    myQueue.Print()
    fmt.Println(myQueue.IsEmpty())

    myQueue.Enqueue(1)
    myQueue.Enqueue(2)
    myQueue.Enqueue(3)

    myQueue.Print()
    myQueue.Dequeue()
    myQueue.Print()
}
```

```
Output:

true
1 2 3
2 3
```

### Priority Queues

That was pretty simple. Now that we understand how normal queues work, we can discuss about priority queues. Priority queues are special types of queues that have different priorities assigned to each items. Imagine you are at an airport, waiting for your flight. The flight attendant notifies the people that they will begin shortly. You hastily pack your belongings and stand first in line. Do you get to board first? Usually no. They will let in veterans and elderly people first, then the first class passengers. You get to board after them.

Priority queues work slightly differently. When you dequeue from a priority queue, the item with the highest priority will be dequeued first. When you call `Front`, you will get an item with the highest priority.

Let's see how we can write this in Go.

```go
type Item struct {
    data     int
    priority int
}

type PriorityQueue struct {
    items []Item
}
```

This is how we define the priority queue and the items inside it. An item holds two information: the actual data being stored, and its priority. Priority queue stores a slice of `Item` objects.

```go
func (pq *PriorityQueue) Sort() {
    sort.Slice(pq.items, func(i, j int) bool {
        return pq.items[i].priority < pq.items[j].priority
    })
}
```

`Sort` is probably the most important method of our priority queue. Like its name suggests, `Sort` sorts the items inside the priority queue in the order of highest priority  to the lowest priority. Priority is represented as an `int` value, and a smaller value means higher priority. 1st, 2nd, 3rd.

`Sort` uses Go's built-in `sort` package. `sort.Slice` takes a slice of any type, and an arbitrary function that determines how to rank items. In this case, we are going to compare the priorities of the i'th and j'th items.

```go
func (pq *PriorityQueue) Enqueue(data Item) {
    pq.items = append(pq.items, data)
    pq.Sort()
}

func (pq *PriorityQueue) Dequeue() {
    pq.Sort()
    pq.items = pq.items[1:]
}

func (pq *PriorityQueue) Front() (Item, error) {
    if len(pq.items) == 0 {
        return Item{}, fmt.Errorf("queue is empty")
    }

    pq.Sort()
    return pq.items[0], nil
}
```

`Enqueue`, `Dequeue` and `Front` stays pretty much the same, except for the fact that they all call `Sort`.

```go
func (pq *PriorityQueue) IsEmpty() bool {
    if len(pq.items) == 0 {
        return true
    } else {
        return false
    }
}

func (pq *PriorityQueue) Print() {
    for _, item := range pq.items {
        fmt.Print(item, " ")
    }
    fmt.Println()
}
```

`IsEmpty` and `Print` stay the same.

Let's try using it!

```go
func main() {
    myQueue := PriorityQueue{[]Item{}}
    myQueue.Print()
    fmt.Println(myQueue.IsEmpty())

    myQueue.Enqueue(Item{0, 1})
    myQueue.Enqueue(Item{1, 4})
    myQueue.Enqueue(Item{2, 3})
    myQueue.Enqueue(Item{3, 0})
    myQueue.Enqueue(Item{4, 2})

    myQueue.Print()
    myQueue.Dequeue()
    myQueue.Print()
}
```

```
Output:

true
{3 0} {0 1} {4 2} {2 3} {1 4} 
{0 1} {4 2} {2 3} {1 4} 
```

Take note of how we added items in an ascending order of 0 to 4, but it is sorted by their priority afterwards.

### Conclusion

Thank you for reading! This was a gentle introduction to queues and priority queues. There are more resources online if you'd like to see more advanced concepts. Hope you learned something along the way!

You can read this post on Medium and my personal site as well.
