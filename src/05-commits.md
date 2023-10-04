# Commitments

Lurk has built-in support for [cryptographic commitments](https://en.wikipedia.org/wiki/Commitment_scheme).

We can create a commitment to any Lurk data with `commit`.

```
user> (commit 123)
[2 iterations] => (comm 0x2937881eff06c2bcc2c8c1fa0818ae3733c759376f76fc10b7439269e9aaa9bc)
```

Now that the Lurk process knows that `(comm 0x2937881eff06c2bcc2c8c1fa0818ae3733c759376f76fc10b7439269e9aaa9bc)` is a commitment to `123`, it can successfully open the commitment.

```
user> (open (comm 0x2937881eff06c2bcc2c8c1fa0818ae3733c759376f76fc10b7439269e9aaa9bc))
[4 iterations] => 123
```

Lurk allows `open` to operate on field elements (so the `(comm ...)` wrapper can be omitted).

```
user> (open 0x2937881eff06c2bcc2c8c1fa0818ae3733c759376f76fc10b7439269e9aaa9bc)
[2 iterations] => 123
```

Because Lurk commitments are based on Poseidon hashes (just as all compound data in Lurk is), it is computationally infeasible to discover a second preimage to the digest represented by a commitment.
This means that Lurk commitments are (computationally) binding.

Lurk also supports explicit hiding commitments.
For when hiding is unimportant, `commit` creates commitments with a default secret of `0`.

```
user> (hide 0 123)
[3 iterations] => (comm 0x2937881eff06c2bcc2c8c1fa0818ae3733c759376f76fc10b7439269e9aaa9bc)
```

However, any field element can be used as the secret, which makes Lurk commitments not only binding but also *hiding*.

```
user> (hide 999 123)
[3 iterations] => (comm 0x3cb2f9666edb8e4444633e960ed56ef0486ea0c3628428680542e2b55f3bf67f)
```

Note that the returned commitment is different from the one returned by both `(commit 123)` and `(hide 0 123)`.
But both commitments open to the same value.

```
user> (= (open 0x2937881eff06c2bcc2c8c1fa0818ae3733c759376f76fc10b7439269e9aaa9bc)
         (open 0x3cb2f9666edb8e4444633e960ed56ef0486ea0c3628428680542e2b55f3bf67f))
[7 iterations] => t
```

## Functional commitments

Again, we can commit to *any* Lurk data, including functions.

```
user> (commit (lambda (x) (+ 7 (* x x))))
[2 iterations] => (comm 0x01ae855385a7e199ab9ec59caa456a9d25b95024dd968145dc7e2327f0cbda9a)
```

The above is a commitment to a function that squares its input then adds seven.

We can construct and evaluate an expression that can only be proven to evaluate to one value: the result of applying the function to a given input, in these examples, 5 and 9.

```
user> ((open 0x01ae855385a7e199ab9ec59caa456a9d25b95024dd968145dc7e2327f0cbda9a) 5)
[12 iterations] => 32
user> ((open 0x01ae855385a7e199ab9ec59caa456a9d25b95024dd968145dc7e2327f0cbda9a) 9)
[12 iterations] => 88
```

Note: Lurk proofs that involve opening a functional commitment and applying it to certain arguments won't reveal any additional information about the function's implementation details or the data it carries, *as long as all of its arguments are consumed*.
If some argument is not consumed, the application will return another function due to auto-currying, revealing its internals.

### Higher-order functional commitments

Higher-order functions are no exceptions and can be committed to in the same manner.

Here, we commit to a function that receives a function as input and applies it to a secret internal value.

```
user> 
(let ((secret-data 222)
      (data-interface (lambda (f) (f secret-data))))
  (commit data-interface))
[7 iterations] => (comm 0x20a1682efabf7b7d83dfe8956b71adb9e47261004cc1e1d612657df425102113)
```

Now we can open it, applying it to a function that adds `111` to the secret value that the committed function hides.

```
user> 
((open 0x20a1682efabf7b7d83dfe8956b71adb9e47261004cc1e1d612657df425102113)
 (lambda (data) (+ data 111)))
[14 iterations] => 333
```

We coin the term Higher-Order Functional Commitments, and as far as we are aware, Lurk is the first and only extant system enabling this powerful usage in the bare language.
