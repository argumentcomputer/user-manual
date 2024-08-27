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
[1 iteration] => <Err NonConstantBuiltin>
```

A list whose first element is neither a built-in operator nor evaluates to a function yields an error when evaluated.

```
user> (1 2 3)
[2 iterations] => <Err ApplyNonFunc>
```

This is because lists are evaluated by first evaluating each element of the list (in order), then treating the first result as a function to be applied to the remaining.
The case above triggers an error because the number `1` does not name a function nor an operator.

## `nil` and `t`

`nil` and `t` are self-evaluating symbols.

```
user> nil
[1 iteration] => nil
user> t
[1 iteration] => t
```

`nil` carries the semantics of "false".

```
user> (= 1 2)
[3 iterations] => nil
user> (if nil 1 2)
[3 iterations] => 2
```

`t` carries the semantics of "true".

```
user> (= 1 1)
[2 iterations] => t
user> (if t 1 2)
[3 iterations] => 1
```

## More on lists

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

And we can left-expand this list by consing an element as its new head.

```
user> (cons 0 (cons 1 nil))
[5 iterations] => (0 1)
```

Thus, within this design, `nil` can be used to represent the empty list.

To deconstruct a pair, we can use `car` and `cdr`, which return the first and the second element respectively.

```
user> (car (cons 1 2))
[4 iterations] => 1
user> (cdr (cons 1 2))
[4 iterations] => 2
```

And in our abstraction of lists, we say that `car` and `cdr` return the list's head and tail respectively.

```
user> (car (cons 0 (cons 1 nil)))
[6 iterations] => 0
user> (cdr (cons 0 (cons 1 nil)))
[6 iterations] => (1)
```

By definition, `(car nil)` and `(cdr nil)` return `nil`.

```
user> (car nil)
[2 iterations] => nil
user> (cdr nil)
[2 iterations] => nil
```

## Functions

A Lurk function can be created by using the `lambda` built-in operator, which requires a list of arguments and a function body.

```
user> (lambda (x) (+ x 1))
[1 iteration] => <Fun (x) (+ x 1)>
```

Then we can write function applications by using lists, as mentioned before.

```
user> ((lambda (x) (+ x 1)) 10)
[6 iterations] => 11
```

Functions with multiple arguments follow the same input design.

```
user> ((lambda (x y) (+ x y)) 3 5)
[7 iterations] => 8
```

Lurk supports partial applications, so we can apply arguments one by one if we want.

```
user> (((lambda (x y) (+ x y)) 3) 5)
[8 iterations] => 8
```

Functions can also be recursive and call themselves by their names.
But how do we name functions?

### Bindings

We'll come back to recursive functions in a bit.
First, let's see how `let` allows us to introduce varible bindings.

```
user> (let ((a 1)) a)
[3 iterations] => 1
```

`let` consumes a list of bindings and the final expression, which can use the introduced variables (or not).

```
user> 
(let ((a 1)
      (b 2)
      (c 3))
  (+ a b))
[7 iterations] => 3
```

When defining the value bound to a variable, we can use the variables that were previously bound.

```
user> 
(let ((a 1)
      (b (+ a 1)))
  b)
[6 iterations] => 2
```

Later bindings shadow previous ones.

```
user>
(let ((a 1)
      (a 2))
  a)
[4 iterations] => 2
```

And inner bindings shadow outer ones.

```
user> 
(let ((a 1))
  (let ((a 2)) a))
[5 iterations] => 2
```

Now we can bind functions to variables.

```
user>
(let ((succ (lambda (x) (+ x 1))))
  (succ 10))
[8 iterations] => 11
```

So can we create a looping recursive function yet?

```
user> 
(let ((loop (lambda (x) (loop x))))
  (loop 0))
[7 iterations] => <Err ApplyNonFunc>
```

In a `let` expression, free variables are expected to be already available by, for instance, being defined on a previous binding.

```
user> 
(let ((not-a-loop (lambda (x) 42))
      (not-a-loop (lambda (x) (not-a-loop x))))
  (not-a-loop 0))
[10 iterations] => 42
```

In the example above, the body of the second binding for `not-a-loop` simply calls the previous definition of `not-a-loop`.

If we want to define recursive functions, we need to use `letrec`.

```
user> 
(letrec ((loop (lambda (x) (loop x))))
  (loop 0))
Error: Loop detected
```

And now we can finally write a recursive function that computes the sum of the first `n` numbers.

```
user> 
(letrec ((sum-upto (lambda (n) (if (= n 0)
                                   0
                                   (+ n (sum-upto (- n 1)))))))
  (sum-upto 5))
[54 iterations] => 15
```

## Higher-order functions

Lurk supports [Higher-order functions](https://en.wikipedia.org/wiki/Higher-order_function).
That is, Lurk functions can receive functions as input and return functions as output, allowing for a wide range of expressive functional programming idioms.

```
user> 
(letrec ((map (lambda (f list)
                (if (eq list nil)
                    nil
                    (cons (f (car list))
                          (map f (cdr list))))))
         (square (lambda (x) (* x x))))
  (map square '(1 2 3 4 5)))
[76 iterations] => (1 4 9 16 25)
```

By the way, how was the list `(1 2 3 4 5)` produced so easily?

## Quoting

The `quote` built-in operator skips the reduction of its argument and returns it *as it is*.

We've seen that trying to reduce `(1 2 3)` doesn't work.
But if we want Lurk not to face that list as a function application, we can use *quoting*.

```
user> (quote (1 2 3))
[1 iteration] => (1 2 3)
```

And we can use `'` as syntax sugar for brevity.

```
user> '(1 2 3)
[1 iteration] => (1 2 3)
```
