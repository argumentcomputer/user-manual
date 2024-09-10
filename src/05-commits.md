# Commitments

Lurk has built-in support for [cryptographic commitments](https://en.wikipedia.org/wiki/Commitment_scheme).

We can create a commitment to any Lurk data with `commit`.

```
lurk-user> (commit 123)
[2 iterations] => #c0x4a902d7be96d1021a473353bd59247ea4c0f0688b5bae0c833a1f624b77ede
```

Now Lurk knows that `#c0x4a902d7be96d1021a473353bd59247ea4c0f0688b5bae0c833a1f624b77ede` is a commitment to `123` and can successfully open it.

```
lurk-user> (open #c0x4a902d7be96d1021a473353bd59247ea4c0f0688b5bae0c833a1f624b77ede)
[2 iterations] => 123
```

Lurk understands it if you use big nums in an `open` expression directly.

```
lurk-user> (open #0x4a902d7be96d1021a473353bd59247ea4c0f0688b5bae0c833a1f624b77ede)
[2 iterations] => 123
```

Because Lurk commitments are based on Poseidon hashes (just as all compound data in Lurk is), it is computationally infeasible to discover a second preimage to the digest represented by a commitment.
This means that Lurk commitments are (computationally) binding.

Lurk also supports explicit hiding commitments.
The hiding *secret* must be a big num.

```
lurk-user> (hide #0x1 123)
[3 iterations] => #c0x2d42d50c445fe7021003b9d177e09e93008f97f74eea0f1c61c3f27aec104f
```

For when hiding is unimportant, `commit` creates commitments with a default secret of `#0x0`.

```
lurk-user> (hide #0x0 123)
[3 iterations] => #0x4a902d7be96d1021a473353bd59247ea4c0f0688b5bae0c833a1f624b77ede
```

And both hashes above open to the same value `123`.

```
lurk-user> (open #0x4a902d7be96d1021a473353bd59247ea4c0f0688b5bae0c833a1f624b77ede)
[2 iterations] => 123
lurk-user> (open #c0x2d42d50c445fe7021003b9d177e09e93008f97f74eea0f1c61c3f27aec104f)
[2 iterations] => 123
```

## Functional commitments

Again, we can commit to *any* Lurk data, including functions.

```
lurk-user> (commit (lambda (x) (+ 7 (* x x))))
[2 iterations] => #c0x8b10a9e88372ee05aea4230b21d7ed9b9e4b1d7f9d2a056d125172ad07c4fc
```

The above is a commitment to a function that squares its input then adds seven.
Then we can open it and apply arguments as usual.

```
lurk-user> ((open #c0x8b10a9e88372ee05aea4230b21d7ed9b9e4b1d7f9d2a056d125172ad07c4fc) 5)
[8 iterations] => 32
lurk-user> ((open #c0x8b10a9e88372ee05aea4230b21d7ed9b9e4b1d7f9d2a056d125172ad07c4fc) 9)
[8 iterations] => 88
```

### Higher-order functional commitments

Higher-order functions are no exceptions and can be committed to in the same manner.

Here, we commit to a function that receives a function as input and applies it to a secret internal value.

```
lurk-user> 
(let ((secret-data 222)
      (data-interface (lambda (f) (f secret-data))))
  (commit data-interface))
[5 iterations] => #c0x759b94bccb5545915690c4e847e0f3e5e42c732eafcc02cbcd55ac50066c9b
```

Now we can open it, applying it to a function that adds `111` to the secret value that the committed function hides.

```
lurk-user>
((open #c0x759b94bccb5545915690c4e847e0f3e5e42c732eafcc02cbcd55ac50066c9b)
 (lambda (data) (+ data 111)))
[10 iterations] => 333
```

We coin the term Higher-Order Functional Commitments, and as far as we are aware, Lurk is the first and only extant system enabling this powerful usage in the bare language.
