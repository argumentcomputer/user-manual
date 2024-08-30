# The REPL

Let's explore more of this tool we've been interacting with, the Lurk REPL.

Until now, every Lurk expression we've evaluated was evaluated under an empty environment.
Trying to evaluate a dangling `a` would trigger an error indicating an unbound variable.

```
user> a
[1 iteration] => <Err UnboundVar>
```

But we can alter the REPL's environment by using what we call *meta commands*.
In particular, we can use the `def` meta command.

```
user> !(def a (+ 1 1))
a
user> a
[1 iteration] => 2
```

We mark meta commands with a `!` before opening parentheses.

There are multiple meta commands available in the REPL.
We're not going to be exhaustive here because there's a `help` meta command.

```
user> !(help)
Available commands:
  assert - Asserts that an expression doesn't reduce to nil.
  assert-emitted - Asserts that the evaluation of an expr emits expected values
  assert-eq - Assert that two expressions evaluate to the same value.
  ...
```

And it can also provide further help on specific meta commands.

```
user> !(help def)
def - Extends env with a non-recursive binding.
  Info:
    Gets macroexpanded to (let ((<symbol> <value>)) (current-env)).
    The REPL's env is set to the result.
  Usage: !(def <symbol> <value>)
  Example:
    !(def foo (lambda () 123))
```

However, we need to go over a few abstractions in order to understand some meta commands you may encounter.

## Chaining callables

A *callable* object is either a function or a commitment to a function.
For example, the following is accepted by Lurk.

```
user> (commit (lambda (x) (+ x 1)))
[2 iterations] => (comm #0x4384e08797d3cd0f911d844e137e54093460c295a4d71cf951259a9ff9712e)
user> (#0x4384e08797d3cd0f911d844e137e54093460c295a4d71cf951259a9ff9712e 10)
[6 iterations] => 11
```

Now let's talk about a specific class of callable objects: ones that can be *chained*.

A chainable callable, once provided with all arguments, must return a pair such that
* The first component is the result of the computation
* The second component is the next callable

To illustrate it, let's define a chainable counter.

```
user>
(commit (letrec ((add (lambda (counter x)
                        (let ((counter (+ counter x)))
                          (cons counter (commit (add counter)))))))
          (add 0)))
[6 iterations] => (comm #0x8ef25bc2228ca9799db65fd2b137a7b0ebccbfc04cf8530133e60087d403db)
```

And let's see what happens when we provide the argument `5` to it.

```
user> (#0x8ef25bc2228ca9799db65fd2b137a7b0ebccbfc04cf8530133e60087d403db 5)
[13 iterations] => (5 . (comm #0x6b2984a56dc2bbb61495c6fdf8076f0debf10ab2ae43aa1de45de64b460141))
```

We get the current counter result and the next callable (a functional commitment in this example).
So let's provide the argument `3` to this next callable.

```
user> (#0x6b2984a56dc2bbb61495c6fdf8076f0debf10ab2ae43aa1de45de64b460141 3)
[13 iterations] => (8 . (comm #0x75711244ea5c5bb0f46692db8851748d860f1e8d7e24a8bbf12caf6ed27c91))
```

The new result is `8`, as expected, and we also get the next callable.
This process can continue indefinitely.

## Lurk protocol
