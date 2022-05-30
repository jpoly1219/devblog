---
title: "Infix to Postfix in Go"
date: 2022-05-29T14:08:22+09:00
draft: false
---

Welcome back to *Introduction to Data Structures in Go*! As I have promised, we will look at a special application of stacks. Specifically, we will explore how to convert infix expressions to their postfix equivalents. Postfix notation may not seem intuitive at first, but after following this post, it will become much clearer.

### A brief primer

"Jacob, it's been a whole week since your last post. I don't remember what these are." Right, let's start with an explanation of what these are.

We learned that expressions were a combination of terms and operators in our math classes, typically written in this format.

```
a + b
```

This is what's known as an *infix* notation. This is because the operator is **in** between the operands.

There are two other notations that we use to evaluate expressions. They are *prefix* and *postfix* notations. Here is a comparison of the three expressions.

```
Infix: <operand><operator><operand>
Prefix: <operator><operand><operand>
Postfix: <operand><operand><operator>
```

The above expression, `a + b`, can be written like this as well:

```
Infix: a+b
Prefix: +ab
Postfix: ab+
```

We use infix notations because it is easy for us to understand. But what is usually easy for us humans is difficult for computers. Imagine writing a program to evaluate an expression. Simple expressions such as `a + b` is easy enough. However, imagine if the expressions get more complicated.

```
{(a*b) + (c*d)} - e
```

When there are a lot of parentheses and operators, you can see how quickly the difficulty increases. Not only do we need to process the operations, but also need to keep track of the order of operations. Because there are nested parentheses, we would have to jump back and forth to evaluate the expressions inside the parentheses first. Keeping track of where we are would be a nightmare.

Now take a look at this.

```
ab*cd*+e-
```

Postfix notation is much easier to follow because we only need to scan the expression from left to right to get our result.

### How do we evaluate a postfix expression?

Let's look at this expression:

```
ab*cd*+e-
a = 2
b = 5
c = 3
d = 1
e = 4
```

We first scan from left to right. We will stop when we see an operator.

```
2 5 * 3 1 * + 4 -
    ^
```

Once we see an operator, we go back two steps. Each time we go back, we keep track of the numbers. Then, we evaluate the expression. In this case, we stop at the multiplication operator `*` and go back two steps. We keep track of the numbers, which are `2` and `5`. To evaluate, all we need to do is `2 * 5`, which yields `10`. The order in which you apply operations is important. For example, if our operator is `-` instead of `*`, then `2 - 5` should yield `-3`, but doing `5 - 2` instead will yield `3`.

```
2 5 * 3 1 * + 4 -
    ^
Numbers: 5, 2
Evaluation: 2 * 5 = 10

Result:
10 3 1 * + 4 -
```

Great! All we need to do now is to repeat the above steps until we are done.

```
10 3 1 * + 4 -
       ^
Numbers: 1, 3
Evaluation: 3 * 1 = 3

Result:
10 3 + 4 -
```

```
10 3 + 4 -
     ^
Numbers: 3, 10
Evaluation: 3 + 10 = 13

Result:
13 4 -
```

```
13 4 -
     ^
Numbers: 4, 13
Evaluation: 13 - 4 = 9

Result:
9
```

We end up with `9`. To double-check, let's calculate the infix expression.

```
Evaluate {(a*b) + (c*d)} - e, given these conditions:
a = 2, b = 5, c = 3, d = 1, e = 4

{(2 * 5) + (3 * 1)} - 4 
= {10 + 3} - 4 
= 13 - 4 
= 9
```

Voila! We have successfully evaluated the postfix expression.

Notice how the `Numbers` we were keeping track of act like a stack? When we see an operator, we backtrack two times and push those numbers into the `Numbers` stack. Then, we pop those numbers and apply the operation.

### Converting it the manual way

We can use stacks to convert infix notations into postfix equivalents. Before we even discuss how to write it in Go, it is important to understand the logic behind the process.

Let's say that we need to convert this expression:

```
a+b*c
```

What we should do is to keep track of operators, similar to how we kept track of the operands in the previous example.

First, we scan from left to right. When we see an operand, we write it to the side. When we see an operator, we add this to our list of operators.

```
a + b * c
^
Operators:
Result: a
```

```
a + b * c
  ^
Operators: +
Result: a
```

```
a + b * c
    ^
Operators: +
Result: ab
```

```
a + b * c
      ^
Operators: +, *
Result: ab
```

Here's the important part.

- When an operator is added, we need to check if the operator we are pointing to takes higher precedence than the operator at the top of our list using order of operations. Here, the operator we are pointing to, `*`, takes precedence over the one at the top of our list, `-`. Multiplication takes precedence over subtraction.

- If it does take precedence, we just add the pointed operator to our list of operators.

- If it doesn't take precedence, however, we pop the list and append the operator in our list to the result string. We continue this until the operator we are pointing to takes precedence over the operator at the top of our list. After all this, we append the operator to the list.

Let's continue.

```
a + b * c
        ^
Operators: +, *
Result: abc
```

Once we reach the end, we pop the list of operators and append the items to our result string, like so:

```
Result: abc*+
```

Let's go over another example.

```
a * b + c
^
Operators:
Result: a
---

a * b + c
  ^
Operators: *
Results: a

---

a * b + c
    ^
Operators: *
Results: ab

---

a * b + c
      ^
Operators: * // check for precedence before you add +!
Results: ab
```

Here, we end up with the second case scenario. Since `+` doesn't take precedence over `*`, we need to pop the list of operators until `+` takes precedence. In this case, we need to pop it until nothing remains on the list.

```
a * b + c
      ^
Operators: + // + is pushed in after * is popped out
Results: ab* // * is popped and appended into the result string
```

The rest is simple.

```
a * b + c
        ^
Operators: +
Results: ab*c

---

Results: ab*c+
```

Let's look at a final example. This time, we will convert an expression with parentheses.

```
{(a*b) + (c*d)} - e
```

Don't worry, we only need to add one more rule.

- If it's an opening parenthesis, we add it to our list of operators.

- If it's a closing parenthesis, we pop our list until a corresponding parenthesis is popped.

- We don't need to append parentheses to our result string. Postfix notations don't require one.

Let's go over step by step!

```
{ ( a * b ) * c - d } - e
    ^
Operators: {, (
Result: a

---

{ ( a * b ) * c - d } - e
      ^
Operators: {, (, *
Result: a

---

{ ( a * b ) * c - d } - e
        ^
Operators: {, (, *
Result: ab

---

{ ( a * b ) * c - d } - e
          ^
Operators: { // pop until the corresponding parenthesis is popped
Result: ab* // * is appended

---

{ ( a * b ) * c - d } - e
            ^
Opeartors: {, *
Result: ab*

---

{ ( a * b ) * c - d } - e
              ^
Operators: {, *
Result: ab*c

---

{ ( a * b ) * c - d } - e
                ^
Operators: {, - // - doesn't take precedence over *
Result: ab*c* // * is popped and appended

---

{ ( a * b ) * c - d } - e
                  ^
Operators: {, -
Result: ab*c*d

---

{ ( a * b ) * c - d } - e
                    ^
Operators: 
Result: ab*c*d-

---

{ ( a * b ) * c - d } - e
                      ^
Operators: -
Result: ab*c*d-

---

{ ( a * b ) * c - d } - e
                        ^
Operators: -
Result: ab*c*d-e

---

Result: ab*c*d-e-
```

It looks complex only because we went over this slowly, step by step. The logic itself is pretty straightforward.

Again, the list of operators acts like a stack, because we are constantly pushing and popping from it.

### Go write it in go

Great! Now that we understand how the high-level logic works, we can start implementing this in Go.

```go
type Stack struct {
    items []float64
}

func (s *Stack) Push(data float64) {
    s.items = append(s.items, data)
}

func (s *Stack) Pop() {
    if s.IsEmpty() {
        return
    }
    s.items = s.items[:len(s.items)-1]
}

func (s *Stack) Top() (float64, error) {
    if s.IsEmpty() {
        return 0.0, fmt.Errorf("stack is empty")
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
}
```

We will use the stack implementation from the last post. This stack can hold `float64` types.

Let's go over how to evaluate a postfix expression first.

#### Evaluating postfix expressions

```go
func evaluatePostfix(exp string) (float64, error) {
    operands := new(Stack)
    chars := strings.Split(exp, " ")

    for _, char := range chars {
        if !isOperator(char) {
            op, err := strconv.ParseFloat(char, 64)
            if err != nil {
                return 0.0, err
            }
            operands.Push(op)
        } else {
            operand2, err := operands.Top()
            if err != nil {
                return 0.0, err
            }
            operands.Pop()

            operand1, err := operands.Top()
            if err != nil {
                return 0.0, err
            }
            operands.Pop()

            calculated, err := calculate(char, operand1, operand2)
            if err != nil {
                return 0.0, err
            }

            operands.Push(calculated)
        }
    }
    result, err := operands.Top()
    if err != nil {
        return 0.0, err
    }

    return result, nil
}
```

We first split the expression into individual characters by space. The expression should be a combination of operands and operators each separated by a space. We can now iterate through the resulting slice of characters.

For each character, we want to check if it is an operand or an operator by using the `isOperator` helper function. If a character is an operand, we push it to the stack.

If the character is an operator, we pop the two topmost items in the stack, saving them as `operand2` and `operand1`. We then evaluate the expression using the `calculate` helper function. Once the calculation is done, we push this to the stack.

After iterating through the expression, we get the topmost item from the stack, which is our final result.

Here are the definitions for the helper functions.

```go
func isOperator(char string) bool {
    switch char {
    case "+":
        return true
    case "-":
        return true
    case "*":
        return true
    case "/":
        return true
    default:
        return false
    }
}

func calculate(operator string, operand1, operand2 float64) (float64, error) {
    result := 0.0
    switch operator {
    case "+":
        result = operand1 + operand2
    case "-":
        result = operand1 - operand2
    case "*":
        result = operand1 * operand2
    case "/":
        result = operand1 / operand2
    default:
        return 0.0, fmt.Errorf("invalid operator")
    }

    return result, nil
}
```

Now we can take a look at how to write the infix-to-postfix conversion function.

#### Convert infix to postfix

We will change our stack so that it stores `string` types instead of `float64` types.

```go
func infixToPostfix(exp string) (string, error) {
    operators := new(StringStack)
    chars := strings.Fields(exp)
    result := ""

    for _, char := range chars {
        if isOpeningParenthesis(char) {
            operators.Push(char)
        } else if isClosingParenthesis(char) {
            for !operators.IsEmpty() && !isMatchingParenthesis(ignoreError(operators.Top()), char) {
                top, err := operators.Top()
                if err != nil {
                    return "", err
                }
                result += top
                operators.Pop()
            }
            operators.Pop()
        } else if !isOperator(char) {
            result += char
        } else if isOperator(char) {
            for !operators.IsEmpty() && hasHigherPrecedence(ignoreError(operators.Top()), char) && !isOpeningParenthesis(ignoreError(operators.Top())) {
                top, err := operators.Top()
                if err != nil {
                    return "", err
                }
                result += top
                operators.Pop()
            }
            operators.Push(char)
        }
    }

    for !operators.IsEmpty() {
        top, err := operators.Top()
        if err != nil {
            return "", err
        }
        result += top

        operators.Pop()
    }

    return result, nil
}
```

The beginning is similar. We create a string slice that holds all the split characters. What's new here is that we need to deal with four cases.

- When the character is an opening parenthesis, we push it to the stack.

- When the character is a closing parenthesis, we pop all the operators until a matching opening parenthesis is popped.

- When the character is an operand, we append it to the result string.

- When the character is an operator, we check for precedence. If the topmost operator in the stack has higher precedence than the character, we pop until an opening parenthesis is found. We then push the character into the stack.

In the end, we flush whatever is left in the stack to the result string.

Here are the definitions for the helper functions.

```go
func isOpeningParenthesis(char string) bool {
    switch char {
    case "(":
        return true
    case "{":
        return true
    case "[":
        return true
    default:
        return false
    }
}

func isClosingParenthesis(char string) bool {
    switch char {
    case ")":
        return true
    case "}":
        return true
    case "]":
        return true
    default:
        return false
    }
}

func isMatchingParenthesis(opening, closing string) bool {
    switch opening {
    case "(":
        if closing == ")" {
            return true
        }
    case "{":
        if closing == "}" {
            return true
        }
    case "[":
        if closing == "]" {
            return true
        }
    default:
        return false
    }
    return false
}

func hasHigherPrecedence(target, source string) bool {
    if (target == "*" || target == "/") && (source == "+" || source == "-") {
        return true
    } else {
        return false
    }
}

func ignoreError(val string, err error) string {
    return val
}
```

Note that ignoring errors isn't a good practice. I used it to deal with annoying error returns because I couldn't pass `operators.Top()` to `hasHigherPrecedence` or `isMatchingParenthesis` due to it returning two values, one of which is an error. If you must ignore errors like this, you could also define a struct method like `stack.TopNoError()` where it will return the topmost value without any error variables.

### Conclusion

Thank you for reading! That was a lengthy post, but I'm proud of you for bearing with me. You usually won't run into situations where you must explicitly convert an infix expression to a postfix one, because the compiler will handle it for you. Nevertheless, it is a great way to learn how stacks work, and I think it's a really interesting topic to know as well. I will come back next week with a write-up on queues. See you then!

You can also read this post on [Medium](https://medium.com/@jpoly1219/stacks-part-2-infix-to-postfix-in-go-610da8776be) and [Dev.to](https://dev.to/jpoly1219/stacks-part-2-infix-to-postfix-in-go-2oam).
