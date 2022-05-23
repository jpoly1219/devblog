---
title: "Stacks in Go"
date: 2022-05-22T14:28:38+09:00
draft: false
---

Welcome back to *Introduction to Data Structures in Go*! Today, we will look at stacks. Stacks are pretty intuitive to understand and have many different uses. It will also come up a lot in coding interviews because its properties can lead to many cool applications.

### What is a stack?

Stacks are data structures that follow the LIFO principle. That is, anything that goes in last will come out first. Imagine a stack of plates. We will stack the plates from bottom to top, then use the top plates first. 

### Write it in Go

The implementation is very simple. We will use slices to implement stacks.

```go
type Stack struct {
    items []int
}
```

We will first define a `Stack` type with `items` as its attribute. Our stack is responsible for holding integers, but you can change the data type to whatever you want it to be.

The two most important methods of stacks are `push` and `pop`. Pushing an item into a stack will add the item to the topmost position, and popping from a stack will remove the topmost item.

```go
func (s *Stack) Push(data int) {
    s.items = append(s.items, data)
}

func (s *Stack) Pop() {
    if s.IsEmpty() {
        return
    }
    s.items = s.items[:len(s.items)-1]
}
```

These methods act on pointers to our `Stack` type, because we need to actively make changes to it.

`Push` is simple enough - we just need to append our data to `s.items`.

`Pop` is simple as well. We first take care of a case where the stack is empty, where the method won't do anything. In other cases, we can take a slice from  `s.items` that includes all but the last element.

We will define three more useful utility methods.

```go
func (s *Stack) Top() (int, error) {
    if s.IsEmpty() {
        return 0, fmt.Errorf("stack is empty")
    }
    return s.items[len(s.items)-1], nil
}

func (s *Stack) IsEmpty() bool {
    if len(s.items) == 0 {
        return true
    }
    return false
}

func (s *Stack) Print() {
    for _, item := range s.items {
        fmt.Print(item, " ")
    }
    fmt.Println()
}
```

`Top` returns the topmost item in the stack. When the stack is empty, it will return a zero-value and an error stating that the stack is empty.

`IsEmpty` returns true if the stack is empty, and false if otherwise.

`Print` iterates through the stack and prints the items.

### This is good and all, but where would I use this?

So far, this just seems like slices with extra steps. Where is this even used?

Stacks are used for these purposes:

- Implementing undo operations.

- Reversing a string or an array.

- Call stack in your memory.

- Navigating forward and backward in browsers and file explorers.

- Converting expressions from and between infix, prefix, and postfix notations.

- Checking whether parentheses are closed properly.

These are all interesting uses, and we can go over the more interesting ones in detail.

#### Undo operation

We just need to store the details of the last operation we have done in a stack. When we want to undo it, we just need to pop the stack to revert to the last saved state.

#### Reversing a string or an array

We iterate over characters in a string or elements in an array and store those in a stack. After we are done, we just need to read the top element of the stack and pop it until the stack is empty.

#### Call stack

The stack in the name *stackoverflow.com* refers to the call stack of a program. When you run a program, a certain amount of memory is allocated to be used as the program's call stack. This is a stack that stores what functions have been called. Each function call stored inside the stack will also hold values of local variables that have been generated inside the function's scope.

FYI, stack overflow is an error that you get when you call more functions than the call stack can hold. This happens commonly when you fail to escape a recursive function. This can also lead to potential attacks such as buffer overflow attacks, where malicious users with high-level privileges can intentionally overflow the stack, causing whatever important data inside to leak out.

#### Infix, prefix, and postfix notations

This is a bit complex. So normally, we would write expressions like this:

```
a + b
```

This is what's known as an *infix* notation. This is because the operator is in between the operands. You can probably guess what the other two are like based on this.

```
Infix: <operand><operator><operand>
Prefix: <operator><operand><operand>
Postfix: <operand><operand><operator>
```

For example, the expression shown above can be written like this as well:

```
Infix: a+b
Prefix: +ab
Postfix: ab+
```

Why on earth would we write something like this? Well, while the infix notation is pleasing to the human eyes, it does little to bring joy to the eyes of a computer. Just like how higher-level languages are human-friendly and lower-level languages are machine-friendly, prefix and postfix notations are easier for computers to process.

Simple expressions are easy enough to parse using infix notations, but this is not scalable. For example, take a look at this expression.

```
{(a*b) + (c*d)} - e
```

When there are a lot of parentheses and operators, you can see how quickly the difficulty increases. Not only do we need to process the operations, but also need to keep track of the order of operations.

```
Infix: {(a*b) + (c*d)} - e
Postfix: ab*cd*+e-
```

Postfix notation is much easier to follow because we only need to scan the expression from left to right to get our result. We can use stacks to convert infix notations into postfix notations. How did we do this? The details will be covered in the next post because it is a bit lengthy.

#### Checking for parentheses closing properly

When a parenthesis is closed properly, we call that *balanced*.

```
Balanced: (a+b), {(a*b) + (c*d)}
Unbalanced: (a+b, []int{0, 1, 2), [)](
```

Manually checking every parenthesis would be tedious. Instead, we can use stacks to check for balance easily.

```go
type StringStack struct {
    items []string
}

func checkBalance(exp string) bool {
    s := StringStack{}

    for _, char := range exp {
        switch char {
        case '(':
            s.Push(string(char))
        case '{':
            s.Push(string(char))
        case '[':
            s.Push(string(char))
        case ')':
            top, _ := s.Top()
            if top == "(" {
                s.Pop()
            } else {
                return false
            }
        case '}':
            top, _ := s.Top()
            if top == "{" {
                s.Pop()
            } else {
                return false
            }
        case ']':
            top, _ := s.Top()
            if top == "[" {
                s.Pop()
            } else {
                return false
            }
        }
    }

    if s.IsEmpty() {
        return true
    }
    return false
}
```

This code looks pretty ugly, but it gets the job done. The code is long because we are handling three different types of parentheses `()`, `{}`, and `[]`. Let's see how the logic works.

- We create an instance of `StringStack` called `s`.

- Iterate through the input string. Each character will be represented as a rune, which is a special way to depict a Unicode character code point. Runes are easily converted to strings.

- If a character is an opening parenthesis, we add it to the stack.

- If a character is a closing parenthesis, we check the topmost element in the stack to see if it matches. For `)`, `s.Top()` should return `(` to balance each other out. If the parentheses are matching, we pop from the stack. If not, the parentheses are not balanced, and we should therefore return `false`.

- Keep going until we are done iterating through the string. If there are any elements left in the stack, this means that some parentheses aren't closed properly, therefore making the parentheses unbalanced. However, if the stack is empty, we return `true` because it is balanced.

### Conclusion

We went over a basic implementation of a stack and its methods in Go. We also covered some of the most popular uses of stacks in the real world. For newcomers, I hope this post helped you understand what stacks are. For seasoned veterans, I hope this post served as a refresher. Regardless, I hope you had fun reading this post. Thank you!

You can also read this post on [Medium](https://medium.com/@jpoly1219/stacks-in-go-f57f4cb87b5f) and [Dev.to](https://dev.to/jpoly1219/stacks-in-go-54k)
