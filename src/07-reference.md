# Language reference

This is a living specification of the current set of built-ins in the `.lurk` package.

## Built-in atoms

### `nil`

`nil` is boolean false and can also represent the empty list.

It has its own type differently from other symbols.

```
lurk-user> (eq nil '())
[3 iterations] => t
lurk-user> (cdr '(1))
[2 iterations] => nil
lurk-user> (if nil "true" "false")
[3 iterations] => "false"
```

### `t`

`t` is boolean true. In boolean contexts like `if`, anything that isn't `nil` is considered false, but `t` is generally used.

```
lurk-user> (eq 1 1)
[2 iterations] => t
lurk-user> (if t "true" "false")
[3 iterations] => "true"
```

## Built-in operators

### `atom`

`(atom x)` returns `nil` if `x` is a pair, and `t` otherwise.

```
lurk-user> (atom nil)
[2 iterations] => t
lurk-user> (atom t)
[2 iterations] => t
lurk-user> (atom (cons 1 2))
[4 iterations] => nil
lurk-user> (atom '(1 2 3))
[2 iterations] => nil
```

### `apply`

`(apply f args)` will call `f` with the argument list `args` and return its result. `args` must be a list.

```
lurk-user> (apply (lambda (x y z) (+ x (+ y z))) '(1 2 3))
[11 iterations] => 6
lurk-user> (apply (lambda (x y z) (+ x (+ y z))) (list 1 2 3))
[11 iterations] => 6
lurk-user> ((apply (lambda (x y z) (+ x (+ y z))) (list 1 2)) 3) ;; partial application also works
[12 iterations] => 6
lurk-user> (apply (lambda (x) x) 1)
[3 iterations] => <Err ArgsNotList>
```

### `begin`

`(begin e1 e2 ... en)` evaluates `e1`, `e2`, ..., `en` and returns the reduced version of `en`.
It is particularly useful when we want to manifest side-effects when evaluating the inner expressions `e1`, `e2`, ...

```
lurk-user> (begin 1 2 3)
[4 iterations] => 3
lurk-user> ((lambda (x) (begin (emit x) (+ x x))) 1)
1
[7 iterations] => 2
```

### `bignum`

`(bignum x)` tries to convert `x` to a big num. Can only be used with commitments currently. Returns `<Err CantCastToBigNum>` if the types are incompatible.

```
lurk-user> (bignum (commit 1))
[3 iterations] => #0xf99d96623838468091ce6590ccb3b829938823d964f3f5962f837f1400e2b
```

### `car`

`(car cons-cell)` returns the first element of its argument if it is a pair. Also works with strings, returning its first character. Returns `<Err NotCons>` otherwise.

```
lurk-user> (car (cons 1 2))
[4 iterations] => 1
lurk-user> (car '(1 2 3))
[2 iterations] => 1
lurk-user> (car (strcons 'a' "b"))
[4 iterations] => 'a'
lurk-user> (car "abc")
[2 iterations] => 'a'
```

### `cdr`

`(cdr cons-cell)` returns the second element of its argument if it is a pair. Also works with strings, returning its tail. Returns `<Err NotCons>` otherwise.

```
lurk-user> (cdr (cons 1 2))
[4 iterations] => 2
lurk-user> (cdr '(1 2 3))
[2 iterations] => (2 3)
lurk-user> (cdr (strcons 'a' "b"))
[4 iterations] => "b"
lurk-user> (cdr "ab")
[2 iterations] => "b"
```

### `char`

`(char x)` tries to convert `x` to a character. Can be used with integers. Returns `<Err CantCastToChar>` if the types are incompatible.

A char can store 32-bits of data, and this conversion truncates 64-bit integers into its 32 least significant bits.

```
lurk-user> (char 0x41)
[2 iterations] => 'A'
lurk-user> (char 65)
[2 iterations] => 'A'
lurk-user> (char nil)
[2 iterations] => <Err CantCastToChar>
```

### `commit`

`(commit x)` is equivalent to `(hide #0x0 x)`. It creates a commitment to the result of evaluating `x`, but sets the secret to the default of zero.

```
lurk-user> (commit 1)
[2 iterations] => #c0xf99d96623838468091ce6590ccb3b829938823d964f3f5962f837f1400e2b
lurk-user> (open #c0xf99d96623838468091ce6590ccb3b829938823d964f3f5962f837f1400e2b)
[3 iterations] => 1
lurk-user> (secret #c0xf99d96623838468091ce6590ccb3b829938823d964f3f5962f837f1400e2b)
[3 iterations] => #0x0
```

### `comm`

`(comm x)` tries to convert `x` to a commitment. Can only be used with big nums currently. Returns `<Err CantCastToComm>` if the types are incompatible.

```
lurk-user> (comm #0x0)
[2 iterations] => #c0x0
lurk-user> (bignum (comm #0x0))
[3 iterations] => #0x0
```

### `cons`

`(cons x y)` creates a new pair with first element `x` and second element `y`. The result of `(cons x y)` is denoted by `(x . y)` or simply `(x)` if `y` is nil.

```
lurk-user> (eq (cons 1 nil) (cons 1 '()))
[6 iterations] => t
lurk-user> (cons 1 2)
[3 iterations] => (1 . 2)
lurk-user> (car (cons 1 2))
[4 iterations] => 1
lurk-user> (cdr (cons 1 2))
[4 iterations] => 2
lurk-user> (cons 1 '(2 3))
[3 iterations] => (1 2 3)
```

### `list`

`(list e1 e2 ... en)` creates a list with the reduced versions of `e1`, `e2`, ..., `en`.

```
lurk-user> (list)
[1 iteration] => nil
lurk-user> (list (+ 1 1) "hi")
[4 iterations] => (2 "hi")
```

### `current-env`

`(current-env)` returns the current environment. The current environment can be modified by `let`, `letrec` and `lambda` expressions.

See also the REPL meta-commands `def`, `defrec` and `clear` for interacting with the current REPL environment.

```
lurk-user> (current-env)
[1 iteration] => <Env ()>
lurk-user> (let ((x 1)) (current-env))
[3 iterations] => <Env ((x . 1))>
lurk-user> (letrec ((x 1)) (current-env))
[3 iterations] => <Env ((x . <Thunk 1>))>
lurk-user> ((lambda (x) (current-env)) 1)
[4 iterations] => <Env ((x . 1))>
```

### `emit`

`(emit x)` prints `x` to the output and returns `x`.

```
lurk-user> (emit 1)
1
[2 iterations] => 1
lurk-user> (let ((f (lambda (x) (emit x)))) (begin (f 1) (f 2) (f 3)))
1
2
3
[16 iterations] => 3
```

### `empty-env`

`(empty-env)` returns the canonical empty environment.

```
lurk-user> (empty-env)
[1 iteration] => <Env ()>
lurk-user> (eq (empty-env) (current-env))
[3 iterations] => t
lurk-user> (let ((x 1)) (eq (empty-env) (current-env)))
[5 iterations] => nil
```

### `eval`

`(eval form env)` evaluates `form` using `env` as its environment. `(eval form)` is equivalent to `(eval form (empty-env))`. Here `form` must be a quoted syntax tree of Lurk code.

```
lurk-user> (eval 1)
[3 iterations] => 1
lurk-user> (eval (+ 1 2)) ;; this expands to `(eval 3)` -- probably not what was intended!
[5 iterations] => 3
lurk-user> (eval '(+ 1 2)) ;; this will actually call `eval` with `'(+ 1 2)` as its argument
[5 iterations] => 3
lurk-user> (eval (cons '+ (cons 1 (cons 2 nil))))
[11 iterations] => 3
lurk-user> (cons '+ (cons 1 (cons 2 nil)))
[7 iterations] => (+ 1 2)
lurk-user> (eval 'x)
[3 iterations] => <Err UnboundVar>
lurk-user> (eval 'x (let ((x 1)) (current-env)))
[6 iterations] => 1
```

### `eq`

`(eq x y)` returns `t` if `x` is equal to `y` and `nil` otherwise.

```
lurk-user> (eq 1 (- 2 1))
[4 iterations] => t
lurk-user> (eq nil nil)
[2 iterations] => t
lurk-user> (eq 'a' 'b')
[3 iterations] => nil
lurk-user> (eq (+ 1 2) (+ 2 1))
[5 iterations] => t
lurk-user> (eq 'a' (char (+ (u64 'A') 32)))
[7 iterations] => t
lurk-user> (eq (cons 1 2) (cons 2 1))
[7 iterations] => nil
```

### `type-eq`

`(type-eq x y)` returns `t` if `x` and `y` have the same type and `nil` otherwise.

```
lurk-user> (type-eq 1 2)
[3 iterations] => t
lurk-user> (type-eq (cons 1 2) (cons 3 4))
[7 iterations] => t
lurk-user> (type-eq 'x 'y)
[3 iterations] => t
lurk-user> (type-eq 'x t)
[3 iterations] => nil
lurk-user> (type-eq 'x nil)
[3 iterations] => nil
lurk-user> (type-eq t nil)
[3 iterations] => nil
lurk-user> (type-eq '() '(1 2)) ;; this is surprisingly the case because `'()` is `nil`, which is a different type from `cons`
[3 iterations] => nil
```

### `type-eqq`

`(type-eqq x y)` returns `t` if `x` and `y` have the same type and `nil` otherwise. The difference is that `x` is treated as an un-evaluated form instead of eagerly evaluating it. This usually means `x` is be some constant value.

```
lurk-user> (type-eqq 1 (+ 1 2))
[4 iterations] => t
lurk-user> (type-eqq (+ 1 2) 1)
[2 iterations] => nil
lurk-user> (type-eqq (+ 1 2) (cons 1 2))
[4 iterations] => t
```

### `hide`

`(hide secret value)` creates a hiding commitment to `value` with `secret` as the salt. `secret` must be a big num.

```
lurk-user> (hide #0x123 456)
[3 iterations] => #c0x4420657169325d52f910f0ffe606b3a7b600a982691926b207f21350120d3d
lurk-user> (open #c0x4420657169325d52f910f0ffe606b3a7b600a982691926b207f21350120d3d)
[3 iterations] => 456
lurk-user> (secret #c0x4420657169325d52f910f0ffe606b3a7b600a982691926b207f21350120d3d)
[3 iterations] => #0x123
```

### `if`

`(if cond then else)` returns `then` if `cond` is non-nil or `else` otherwise. Only `nil` values are considered false. `(if cond then)` is equivalent to `(if cond then nil)`

```
lurk-user> (if t "true" "false")
[3 iterations] => "true"
lurk-user> (if nil "true" "false")
[3 iterations] => "false"
lurk-user> (if nil "true")
[2 iterations] => nil
lurk-user> (if 0 "true") ;; note: `0` is *not* `nil`
[3 iterations] => "true"
```

### `lambda`

`(lambda (args...) body)` creates a function that takes a list of arguments and returns the result of evaluating `body` by binding the variable bindings to the supplied arguments. A function is called when it is in the head of a list.

The list of arguments can optionally end with `&rest <var>`, which denotes that any remaining arguments, if present, are bound to `<var>` as a list.

```
lurk-user> (lambda () 1)
[1 iteration] => <Fun () 1>
lurk-user> (lambda (x) x)
[1 iteration] => <Fun (x) x>
lurk-user> ((lambda () 1)) 
[3 iterations] => 1
lurk-user> ((lambda (x) x) 1)
[4 iterations] => 1
lurk-user> ((lambda (x y z) (+ x (+ y z))) 1 2 3)
[10 iterations] => 6
lurk-user> (let ((f (lambda (x y z) (+ x (+ y z))))) (+ (f 1 2 3) (f 3 2 1))) ;; here `f` is the variable bound to the `lambda`
[19 iterations] => 12
lurk-user> ((lambda (x y &rest z) z) 1 2 3)
[6 iterations] => (3)
lurk-user> ((lambda (x y &rest z) z) 1 2)
[5 iterations] => nil
lurk-user> ((lambda (&rest x) x))
[3 iterations] => nil
```

### `let`

`(let ((var binding)...) body)` extends the current environment with a set of variable bindings and then evaluate `body` in the updated environment. `let` is used for binding values to names and modifying environments. See also the `def` REPL meta command.

```
lurk-user> (let ((x 1) (y 2)) (+ x y))
[6 iterations] => 3
lurk-user> (let ((x 1) (y 2)) (current-env))
[4 iterations] => <Env ((y . 2) (x . 1))>
```

### `letrec`

`(letrec ((var binding)...) body)` is similar to `let`, but it enables recursion by allowing references to `var` inside its own `binding`. Generally, the binding is be a `lambda` expression representing a recursive function. See also the `defrec` REPL meta command.

```
lurk-user> (letrec ((x 1)) x)
[3 iterations] => 1
lurk-user> (letrec ((x 1)) (current-env))
[3 iterations] => <Env ((x . <Thunk 1>))> ;; Thunks are the internal representation used for recursive evaluation
lurk-user> (letrec ((last (lambda (x) (if (cdr x) (last (cdr x)) (car x))))) (last '(1 2 3)))
[19 iterations] => 3
```

### `u64`

`(u64 x)` tries to convert `x` to a 64-bit unsigned integer. Can be used with characters. Returns `<Err CantCastToU64>` if the types are incompatible.

```
lurk-user> (u64 'A')
[2 iterations] => 65
lurk-user> (u64 100) ;; this is a no-op
[2 iterations] => 100
lurk-user> (u64 nil)
[2 iterations] => <Err CantCastToU64>
```

### `open`

`(open x)` opens the commitment `x` and return its value. If `y` is a big num, then `(open y)` is equivalent to `(open (comm y))`.

```
lurk-user> (open (commit 1))
[3 iterations] => 1
lurk-user> (open #c0xf99d96623838468091ce6590ccb3b829938823d964f3f5962f837f1400e2b)
[2 iterations] => 1
lurk-user> (open #0xf99d96623838468091ce6590ccb3b829938823d964f3f5962f837f1400e2b)
[2 iterations] => 1
```

### `quote`

`(quote x)` returns `x` as its un-evaluated syntax form. `(quote x)` is equivalent to `'x`.

```
lurk-user> (quote 1)
[1 iteration] => 1
lurk-user> (quote (+ 1 2))
[1 iteration] => (+ 1 2)
lurk-user> (quote 1)
[1 iteration] => 1
lurk-user> (quote (+ 1 2))
[1 iteration] => (+ 1 2)
lurk-user> '(+ 1 2)
[1 iteration] => (+ 1 2)
lurk-user> (cdr (quote (+ 1 2)))
[2 iterations] => (1 2)
lurk-user> (eval (cdr '(- + 1 2)))
[6 iterations] => 3
```

### `secret`

`(secret x)` opens the commitment `x` and return its secret. If `y` is a big num, then `(secret y)` is equivalent to `(secret (comm y))`.

```
lurk-user> (secret (commit 1))
[3 iterations] => #0x0
lurk-user> (secret (hide #0x123 2))
[4 iterations] => #0x123
lurk-user> (secret #c0x76cc74360d9492188f072ecdafd2ddf50fadac0ce2007c551c09fdcd556109)
[2 iterations] => #0x123
lurk-user> (secret #0x76cc74360d9492188f072ecdafd2ddf50fadac0ce2007c551c09fdcd556109)
[2 iterations] => #0x123
```

### `strcons`

`(strcons x y)` creates a new string with first element `x` and rest `y`. `x` must be a character and `y` must be a string. Strings are represented as lists of characters, but because the type of strings and lists are different, `strcons` is used for constructing a string instead.

```
lurk-user> (strcons 'a' nil) ;; the empty list is not the same as the empty string
[3 iterations] => <Err NotString>
lurk-user> (strcons 'a' "")
[3 iterations] => "a"
lurk-user> (strcons 'a' "bc")
[3 iterations] => "abc"
lurk-user> (car (strcons 'a' "bc"))
[4 iterations] => 'a'
lurk-user> (cdr (strcons 'a' "bc"))
[4 iterations] => "bc"
lurk-user> (cons 'a' "bc")
[3 iterations] => ('a' . "bc") ;; note how the cons is not the same thing as "abc"
```

### `breakpoint`

`(breakpoint)` places a debugger breakpoint at the current evaluation state and returns `nil`.
`(breakpoint x)` places a debugger breakpoint and returns `x`, reduced.

`!(help debug)` for more information on the native debugger.

### `fail`

`(fail)` errors out from evaluation, unprovably.

## Built-in numeric operators

### `+`

`(+ a b)` returns the sum of `a` and `b`. Overflow can happen implicitly. Returns `<Err ArgNotNumber>` if the types are not compatible.

```
lurk-user> (+ 1 2)
[3 iterations] => 3
lurk-user> (+ 1n 2n)
[3 iterations] => 3n
lurk-user> (+ #0x1 #0x2) ;; no big num arithmetic yet
[3 iterations] => <Err ArgNotNumber>
lurk-user> (+ 18446744073709551615 18446744073709551615)
[2 iterations] => 18446744073709551614 ;; implicit overflow for u64
```

### `-`

`(- a b)` returns the difference between `a` and `b`. Underflow can happen implicitly. Returns `<Err ArgNotNumber>` if the types are not compatible.

```
lurk-user> (- 2 1)
[3 iterations] => 1
lurk-user> (- 2n 1n)
[3 iterations] => 1n
lurk-user> (- 0 1)
[3 iterations] => 18446744073709551615
lurk-user> (- 0n 1n)
[3 iterations] => 2013265920n
```

### `*`

`(* a b)` returns the product of `a` and `b`. Overflow can happen implicitly. Returns `<Err ArgNotNumber>` if the types are not compatible.

```
lurk-user> (* 2 3)
[3 iterations] => 6
lurk-user> (* 2n 3n)
[3 iterations] => 6n
lurk-user> (* 18446744073709551615 18446744073709551615)
[2 iterations] => 1
```

### `/`

`(/ a b)` returns the quotient between `a` and `b`. Returns `<Err ArgNotNumber>` if the types are not compatible. Returns `<Err DivByZero>` if `b` is zero.

When `a` and `b` are integers, the fractional part of the result is truncated and the result is the usual integer division.

When `a` and `b` are native field elements, `(/ a b)` returns the field element `c` such that `(eq a (* c b))`. In other words, `(/ 1n x)` gives the multiplicative inverse of the native field element `x`.

```
lurk-user> (/ 10 2)
[3 iterations] => 5
lurk-user> (/ 9 2)
[3 iterations] => 4
lurk-user> (/ 10n 2n)
[3 iterations] => 5n
lurk-user> (/ 9n 2n)
[3 iterations] => 1006632965n
lurk-user> (* 1006632965n 2n)
[3 iterations] => 9n
lurk-user> (* (/ 9 2) 2)
[4 iterations] => 8
lurk-user> (/ 1 0)
[3 iterations] => <Err DivByZero>
```

### `%`

`(% a b)` returns the remainder of dividing `a` by `b`. Returns `<Err NotU64>` if the arguments are not unsigned integers.

```
lurk-user> (% 15 7)
[3 iterations] => 1
lurk-user> (% 15 6)
[3 iterations] => 3
lurk-user> (/ 15 6)
[3 iterations] => 2
lurk-user> (+ (* 2 6) 3)
[5 iterations] => 15
```

### `=`

`(= a b)` returns `t` if `a` and `b` are equal and `nil` otherwise. The arguments must be numeric. Returns `<Err ArgNotNumber>` if the types are not compatible.

```
lurk-user> (= 123 123)
[2 iterations] => t
lurk-user> (= 123n 123n)
[2 iterations] => t
lurk-user> (= #0x123 #0x123)
[2 iterations] => t
lurk-user> (= "abc" "abc")
[2 iterations] => <Err ArgNotNumber>
```

### `<`

`(< a b)` returns `t` if `a` is strictly less than `b` and `nil` otherwise. The arguments must be numeric. Returns `<Err ArgNotNumber>` if the types are not compatible. Note that native field elements cannot be compared like this.

```
lurk-user> (< "a" "b")
[3 iterations] => <Err ArgNotNumber>
lurk-user> (< 1n 2n)
[3 iterations] => <Err NotU64>
lurk-user> (< #0x123 #0x456)
[3 iterations] => t
lurk-user> (< 123 456)
[3 iterations] => t
```

### `>`

`(> a b)` returns `t` if `a` is strictly greater than `b` and `nil` otherwise. The arguments must be numeric. Returns `<Err ArgNotNumber>` if the types are not compatible. Note that native field elements cannot be compared like this.

```
lurk-user> (> #0x123 #0x456)
[3 iterations] => nil
lurk-user> (> 456 123)
[3 iterations] => t
```

### `<=`

`(<= a b)` returns `t` if `a` is less than or equal to `b` and `nil` otherwise. The arguments must be numeric. Returns `<Err ArgNotNumber>` if the types are not compatible. Note that native field elements cannot be compared like this.

```
lurk-user> (<= #0x123 #0x456)
[3 iterations] => t
lurk-user> (<= 123 123)
[2 iterations] => t
```

### `>=`

`(>= a b)` returns `t` if `a` is greater than or equal to `b` and `nil` otherwise. The arguments must be numeric. Returns `<Err ArgNotNumber>` if the types are not compatible. Note that native field elements cannot be compared like this.

```
lurk-user> (>= #0x123 #0x456)
[3 iterations] => nil
lurk-user> (>= 123 123)
[2 iterations] => t
```
