# Lurk is a Lisp

Since Lurk is a Lisp, it has practically no syntax and expressions can be formed with simple rules.

The expression `3` is self-evaluating.

```
user> 3
[1 iteration] => 3
```

When an expression is evaluated, you should see a response with the result and an iteration count, in this case, 1.
You can think of this as representing the "cost" or number of "clock cycles" required to evaluate the expression.
In the simplest case of a self-evaluating expression, only a single iteration is required.

Characters and strings are also self-evaluating.

```
user> 'a'
[1 iteration] => 'a'
user> "abc"
[1 iteration] => "abc"
```

Lists are evaluated by treating the first element as a potential function (more on functions later) and the rest as its arguments.

```
user> (+ 2 3)
[3 iterations] => 5
```

Technically, `+` is a built-in operator: it does not evaluate to a Lurk function but behaves like one here.

Built-in operators cannot be independently evaluated.

```
user> +
Evaluation encountered an error after 1 iteration
```

A list whose first element is neither a built-in operator nor evaluates to a function yields an error when evaluated.

```
user> (1 2 3)
Evaluation encountered an error after 1 iteration
```

This is because lists are evaluated by first evaluating each element of the list (in order), then treating the first result as a function to be applied to the remaining.
The case above triggers an error because the number `1` does not name a function nor an operator.

### `nil` and `t`

`nil` and `t` are self-evaluating symbols.

```
user> nil
[1 iteration] => nil
user> t
[1 iteration] => t
```

`nil` carries the semantics of "false".

```
user> (eq 1 2)
[3 iterations] => nil
user> (if nil 1 2)
[3 iterations] => 2
```

`t` carries the semantics of "true".

```
user> (eq 1 1)
[3 iterations] => t
user> (if t 1 2)
[3 iterations] => 1
```

### More on lists

Pairs are constructed with `cons`, and their elements are separated by a `.` when printed.

```
user> (cons 1 2)
[3 iterations] => (1 . 2)
```

A pair whose second element is `nil` is said to form a list with one element.

```
user> (cons 1 nil)
[3 iterations] => (1)
```

And we can left-expand this list by consing a new element as its head.

```
user> (cons 0 (cons 1 nil))
[6 iterations] => (0 1)
```

Thus, within this design, `nil` can be used as the empty list.

One might think that a list is a basic datum that should be self-evaluating.
But remember that a list, when subject to evaluation, carries the semantic of a function application.
So when we say `(cons 1 nil)`, we're inputting a list with three elements: the symbol `cons`, the number `1` and the symbol `nil`.
Lurk then interprets this as a call to the `cons` built-in operator, with arguments `1` and `nil`, finally producing the list `(1)`.

There is an entire chapter dedicated to built-in operators. For now, it's enough to know how Lurk expressions are formed.
