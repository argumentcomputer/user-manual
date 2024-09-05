# Commitments

Lurk has built-in support for [cryptographic commitments](https://en.wikipedia.org/wiki/Commitment_scheme).

We can create a commitment to any Lurk data with `commit`.

```
user> (commit 123)
[2 iterations] => #c0x3719f5d02845123a80da4f5077c803ba0ce1964e08289a9d020603c1f3c450
```

Now Lurk knows that `#c0x3719f5d02845123a80da4f5077c803ba0ce1964e08289a9d020603c1f3c450` is a commitment to `123` and can successfully open it.

```
user> (open #c0x3719f5d02845123a80da4f5077c803ba0ce1964e08289a9d020603c1f3c450)
[2 iterations] => 123
```

Note that `comm` is a built-in operator that coerces big nums into commitments.
But Lurk understands it if you use big nums in an `open` expression directly.

```
user> (open #0x3719f5d02845123a80da4f5077c803ba0ce1964e08289a9d020603c1f3c450)
[2 iterations] => 123
```

Because Lurk commitments are based on Poseidon hashes (just as all compound data in Lurk is), it is computationally infeasible to discover a second preimage to the digest represented by a commitment.
This means that Lurk commitments are (computationally) binding.

Lurk also supports explicit hiding commitments.
The hiding *secret* must be a big num.

```
user> (hide #0x1 123)
[3 iterations] => #c0x7e14ff1d0f6c3844f57c9120786d54256e73c8f08a80f31d6502d87884b3d4
```

For when hiding is unimportant, `commit` creates commitments with a default secret of `#0x0`.

```
user> (hide #0x0 123)
[3 iterations] => #c0x3719f5d02845123a80da4f5077c803ba0ce1964e08289a9d020603c1f3c450
```

And both hashes above open to the same value `123`.

```
user> (open #0x3719f5d02845123a80da4f5077c803ba0ce1964e08289a9d020603c1f3c450)
[2 iterations] => 123
user> (open #0x7e14ff1d0f6c3844f57c9120786d54256e73c8f08a80f31d6502d87884b3d4)
[2 iterations] => 123
```

## Functional commitments

Again, we can commit to *any* Lurk data, including functions.

```
user> (commit (lambda (x) (+ 7 (* x x))))
[2 iterations] => #c0x84a0ebe63fadc8a5e8b848a644a4af72150b6c11652cdbe862f2ee3ffce614
```

The above is a commitment to a function that squares its input then adds seven.
Then we can open it and apply arguments as usual.

```
user> ((open #0x84a0ebe63fadc8a5e8b848a644a4af72150b6c11652cdbe862f2ee3ffce614) 5)
[8 iterations] => 32
user> ((open #0x84a0ebe63fadc8a5e8b848a644a4af72150b6c11652cdbe862f2ee3ffce614) 9)
[8 iterations] => 88
```

### Higher-order functional commitments

Higher-order functions are no exceptions and can be committed to in the same manner.

Here, we commit to a function that receives a function as input and applies it to a secret internal value.

```
user> 
(let ((secret-data 222)
      (data-interface (lambda (f) (f secret-data))))
  (commit data-interface))
[5 iterations] => #c0x4668b9badf58209537dbb62e132badc5bb7bbaf137a8daeeef550046634da8
```

Now we can open it, applying it to a function that adds `111` to the secret value that the committed function hides.

```
user>
((open #0x4668b9badf58209537dbb62e132badc5bb7bbaf137a8daeeef550046634da8)
 (lambda (data) (+ data 111)))
[10 iterations] => 333
```

We coin the term Higher-Order Functional Commitments, and as far as we are aware, Lurk is the first and only extant system enabling this powerful usage in the bare language.
