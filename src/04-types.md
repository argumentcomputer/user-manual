# Data types

Even though Lurk is not statically typed, it is strongly typed.

### Unsigned integers

The default numeric type in Lurk is `u64`, which roundtrips as one would expect:

```
lurk-user> (- 0 1)
[3 iterations] => 18446744073709551615
lurk-user> (+ 18446744073709551615 1)
[3 iterations] => 0
```

### Finite field elements

Lurk also allows operating over finite field elements directly, which is cheaper than dealing with `u64` in terms of proving costs.

```
lurk-user> (+ 1n 2n)
[3 iterations] => 3n
```

Beware, though, that different finite fields have different sizes.
The current Lurk instantiation uses [Baby Bear](https://github.com/Plonky3/Plonky3/blob/971d7b19b3284c3aeaa185c3e0b81ad03a1de64e/baby-bear/src/baby_bear.rs#L7-L8).

```
lurk-user> (- 0n 1n)
[3 iterations] => 2013265920n
```

So, to write stable Lurk code, use finite field elements only if you are sure that those values won't grow too much.

### Big nums

Numbers large enough to represent the domain of approximately 256-bit hash digests.

```
lurk-user> #0x1
[1 iteration] => #0x1
lurk-user> #0x4668b9badf58209537dbb62e132badc5bb7bbaf137a8daeeef550046634da8
[1 iteration] => #0x4668b9badf58209537dbb62e132badc5bb7bbaf137a8daeeef550046634da8
```

### Commitments

Actual cryptographic commitments to Lurk data, as we shall see in the next chapter.
It has the same domain range of big nums.

```
lurk-user> (comm #0x0)
[2 iterations] => #c0x0
```

### Characters

Lurk uses [UTF-8](https://en.wikipedia.org/wiki/UTF-8) encoding to represent characters internally.
Meaning that a wide range of characters are accepted.

```
lurk-user> 'a'
[1 iteration] => 'a'
lurk-user> '€'
[1 iteration] => '€'
lurk-user> 'ệ'
[1 iteration] => 'ệ'
```

### Strings

Very briefly, strings are sequences of characters.
We can even extract the head and the tail of a string.

```
lurk-user> (car "abc")
[2 iterations] => 'a'
lurk-user> (cdr "abc")
[2 iterations] => "bc"
```

### Cons (pair)

We've already covered pairs, but it's important to mention that pairs are of their own type.

```
lurk-user> (type-eq (cons 1 2) (cons 'a' nil))
[7 iterations] => t
```

### Symbols

These are the typical language entities we use to bind values to, as in `(let ((a 1)) a)`.

```
lurk-user> (type-eq 'a 'x)
[3 iterations] => t
```

`t` and `nil` are of this same type.

```
lurk-user> (type-eq t 'x)
[3 iterations] => t
lurk-user> (type-eq t nil)
[3 iterations] => t
```

### Keywords

Keywords are self-evaluating symbols without special semantics like `t` or `nil`.

```
lurk-user> :foo
[1 iteration] => :foo
lurk-user> (type-eq :foo :bar)
[3 iterations] => t
```

### Built-ins

These are symbols in the `.lurk.builtin` package (more on packages later).

```
lurk-user> (type-eq 'cons 'let)
[3 iterations] => t
lurk-user> (type-eq 'cons 'x)
[3 iterations] => nil
```

### Functions

Functions are the result of evaluated `lambda` expressions.

```
lurk-user> (type-eq (lambda () nil) (lambda (x) (+ x 1)))
[3 iterations] => t
```

Undersaturated functions, that is, functions that have not been supplied all of their arguments, are still of the same type.

```
lurk-user>
(type-eq
  (lambda (x) (+ x 1))
  ((lambda (x y) (+ x y)) 1))
[5 iterations] => t
```

### Environments

Environments carry pairs of symbol/value bindings.
Every expression evaluation in Lurk happens with respect to a certain environment.

The starting environment in the REPL is empty by default.

```
lurk-user> (current-env)
[1 iteration] => <Env ()>
```

The environment grows as we create new bindings.

```
lurk-user> (let ((a 1) (b 2)) (current-env))
[4 iterations] => <Env ((b . 2) (a . 1))>
```

Later, we will learn how to change the REPL's environment.
