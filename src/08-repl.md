# The REPL

Let's explore more of this tool we've been interacting with, the Lurk REPL.

## Meta commands

Until now, every Lurk expression we've evaluated was evaluated under an empty environment.
Trying to evaluate a dangling `a` would trigger an error indicating an unbound variable.

```
lurk-user> a
[1 iteration] => <Err UnboundVar>
```

But we can alter the REPL's environment by using what we call *meta commands*.
In particular, we can use the `def` meta command.

```
lurk-user> !(def a (+ 1 1))
a
lurk-user> a
[1 iteration] => 2
```

We mark meta commands with a `!` before opening parentheses.

There are multiple meta commands available in the REPL.
We're not going to be exhaustive here because there's a `help` meta command.

```
lurk-user> !(help)
Available commands:
  assert - Asserts that an expression doesn't reduce to nil.
  assert-emitted - Asserts that the evaluation of an expr emits expected values
  assert-eq - Assert that two expressions evaluate to the same value.
  ...
```

And it can also provide further help on specific meta commands.

```
lurk-user> !(help def)
def - Extends env with a non-recursive binding.
  Info:
    Gets macroexpanded to (let ((<symbol> <expr>)) (current-env)).
    The REPL's env is set to the result.
  Format: !(def <symbol> <expr>)
  Example:
    !(def foo (lambda () 123))
  Returns: The binding symbol
```

As hinted above, meta commands have return values.
That's because we can build expressions that contain meta commands.
The meta commands will be evaluated first, having their occurrences in the original expression replaced by their respective return values.
The resulting expression is then evaluated.

```
lurk-user> (+ !(def b 21) b)
b
[2 iterations] => 42
```

When processing `(+ !(def b 21) b)`, `!(def b 21)` is evaluated, which adds `b` to the environment and returns the symbol `b`.
The resulting expression is `(+ b b)`, which, once evaluated, results in `42`.

To double check which expression was evaluated, you can enter the debug mode with `!(debug)` (make sure to check `!(help debug)`!).

This is what the debug mode shows:

```
?0: (+ b b)
?1: b
 1: b ↦ 21
!1: b ↦ 21
 0: (+ b b) ↦ 42
```

Now, let's go over a few abstractions in order to understand some meta commands you will encounter.

## Chaining callables

A *callable* object is either a function or a commitment to a function.
For example, the following is accepted by Lurk.

```
lurk-user> (commit (lambda (x) (+ x 1)))
[2 iterations] => #c0x1f4971f046cd9b037eadbc3fcde177431a712e8cef82fcec9e010fc5a65891
lurk-user> (#c0x1f4971f046cd9b037eadbc3fcde177431a712e8cef82fcec9e010fc5a65891 10)
[6 iterations] => 11
```

Now let's talk about a specific class of callable objects: ones that can be *chained*.

A chainable callable, once provided with all arguments, must return a pair such that
* The first component is the result of the computation
* The second component is the next callable

To illustrate it, let's define a chainable counter.

```
lurk-user>
(commit (letrec ((add (lambda (counter x)
                        (let ((counter (+ counter x)))
                          (cons counter (commit (add counter)))))))
          (add 0)))
[7 iterations] => #c0x64fee21bad514ff18399dfc5066caebf34acc0441c9af675ba95a998077591
```

And let's see what happens when we provide the argument `5` to it.

```
lurk-user> (#c0x64fee21bad514ff18399dfc5066caebf34acc0441c9af675ba95a998077591 5)
[14 iterations] => (5 . #c0x73c6bd6ffb74c34b7c21e83aaaf71ddf919bb2a9c93ecb43c656f8dc67060a)
```

We get the current counter result and the next callable (a functional commitment in this example).
So let's provide the argument `3` to this next callable.

```
lurk-user> (#c0x73c6bd6ffb74c34b7c21e83aaaf71ddf919bb2a9c93ecb43c656f8dc67060a 3)
[14 iterations] => (8 . #c0x67dfd6673beec5f6758e3404d74af882f66b1084adabb97bc27cba554def07)
```

The new result is `8` and we also get the next callable, as expected.
This process can continue indefinitely.

The background provided above should be enough to understand the `chain` and `transition` meta commands.

## Lurk protocol

The Lurk protocol API is useful when a verifier party wants to be specific about what a proof must claim before checking whether it verifies or not.

Broadly speaking, a Lurk protocol is an open declaration for how the public input of a proof will be constructed.

A claim of a Lurk reduction states that an expression `expr`, when evaluated under an environment `env`, results on `res`.
Shortly, `(expr, env) -> res`.
Or, using pairs, `((expr . env) . res)`.

A protocol may require arguments in order to construct the expected claim.
And a protocol may also reject a proof by returning `nil` instead of a claim.

The last bit of information before we go over a simple example is that a protocol can also define a post-verification predicate.
That's a 0-arg function that uses the protocol arguments to validate a claim *after* verification happens.
This is useful when there's a check that needs to happen that's more expensive than verifying the proof.
Such a check would fit well as a post-verification predicate.

Now let's write a protocol that receives a number `n` and proves that `n = n` under the empty environment.
This shouldn't be a hard task for any prover who accepts the challenge, but it's a good starting example.

```
lurk-user>
!(defprotocol simple-protocol (n)
  (cons
    ;; claim definition
    (cons (cons (list '= n n) (empty-env))
          t)
    ;; post-verification predicate is not provided
    nil)
  :description "(= n n) reduces to t")
```

`(list '= n n)` creates the Lurk expression `(= <n> <n>)` on the fly.
More on this later.

That protocol can be persisted on the file system and shared.

```
lurk-user> !(dump-expr simple-protocol "simple-protocol-file")
Data persisted on file `simple-protocol-file`
```

Some prover out there can download the protocol and load it from their file system.

```
lurk-user> !(defq simple-protocol !(load-expr "simple-protocol-file"))
simple-protocol
```

And then prove it for, say, the number `3`.

```
lurk-user> !(prove-protocol simple-protocol "protocol-proof" 3)
Proof key: "56b674aa77c68bdc916a01c122da092f7a5fdc9fd0b13646382f80bebcbe99"
Protocol proof saved on file `protocol-proof`
```

Let's inspect the (cached) proof we've just created.

```
lurk-user> !(inspect "56b674aa77c68bdc916a01c122da092f7a5fdc9fd0b13646382f80bebcbe99")
Expr: (= 3 3)
Env: <Env ()>
Result: t
```

Now the prover sends the `protocol-proof` file to the verifier, who can verify it.

```
lurk-user> !(verify-protocol simple-protocol "protocol-proof")
Proof accepted by the protocol
```

In the example above, the post-verification predicate was not defined (set to `nil`).
Also, there was no restriction on the number used by the prover.
A slight variation would be to require that the number must be an `u64`.

```
lurk-user>
!(defprotocol simple-protocol (n)
  (cons
    (if (type-eqq 0 n)
      ;; return a claim if n is indeed u64
      (cons (cons (list '= n n) (empty-env))
            t)
      ;; the following nil makes the protocol reject the proof
      nil)
    ;; again, no post-verification predicate
    nil)
  :description "(= n n) reduces to t")
```

And if some prover tries to prove it with, say, a field element instead of an `u64`, the REPL won't allow it.

```
lurk-user> !(prove-protocol simple-protocol "protocol-proof" 3n)
!Error: Pre-verification predicate rejected the input
```

The same check happens on the verifier side (of course!), which would be able to reject a proof generated with a maliciously edited REPL on the prover side.

## Sharing proofs

The proof files created with the `prove` meta command showcase Lurk's proving capabilities, but they may contain information that we don't want to disclose.

```
lurk-user> !(def password "some password")
password
lurk-user> !(def hash (hide (bignum (commit password)) "private data"))
hash
lurk-user> !(prove (begin (open hash) t))
[4 iterations] => t
Proof key: "6e45ffb851078d6abf90a5d9b434ae34f2aad64d0e5880ca9c0189737f4a9e"
lurk-user> !(inspect "6e45ffb851078d6abf90a5d9b434ae34f2aad64d0e5880ca9c0189737f4a9e")
Expr: (begin (open hash) t)
Env: <Env ((hash . #c0x145c99908adac10ee79eb5e1d74ad8fe42c10abe79f5ef3d5dbb96915cc7a3) (password . "some password"))>
Result: t
```

See above that the binding for `password` is visible in the claim's environment.
Thus, such proof files are not meant to be shared.

**The safe way to share proofs** is using the protocol API mentioned above, which creates proof files *designed* to be shared.
This is because protocol proofs carry just enough information for the verifier to reconstruct the entire claim, and do not include unnecessary (and potentially private) information from the prover's REPL environment.
