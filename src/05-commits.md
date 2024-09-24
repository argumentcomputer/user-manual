# Commitments

Lurk has built-in support for [cryptographic commitments](https://en.wikipedia.org/wiki/Commitment_scheme).

We can create a commitment to any Lurk data with `commit`.

```
lurk-user> (commit 123)
[2 iterations] => #c0x944834111822843979ace19833d05ca9daf2f655230faec517433e72fe777b
```

Now Lurk knows that `#c0x944834111822843979ace19833d05ca9daf2f655230faec517433e72fe777b` is a commitment to `123` and can successfully open it.

```
lurk-user> (open #c0x944834111822843979ace19833d05ca9daf2f655230faec517433e72fe777b)
[2 iterations] => 123
```

Lurk understands it if you use big nums in an `open` expression directly.

```
lurk-user> (open #0x944834111822843979ace19833d05ca9daf2f655230faec517433e72fe777b)
[2 iterations] => 123
```

Because Lurk commitments are based on Poseidon hashes (just as all compound data in Lurk is), it is computationally infeasible to discover a second preimage to the digest represented by a commitment.
This means that Lurk commitments are (computationally) binding.

Lurk also supports explicit hiding commitments.
The hiding *secret* must be a big num.

```
lurk-user> (hide #0x1 123)
[3 iterations] => #c0x483cd4ed61cb38d4722743fe470c4c81abf1a568ceae9423864d5acf03739f
```

For when hiding is unimportant, `commit` creates commitments with a default secret of `#0x0`.

```
lurk-user> (hide #0x0 123)
[3 iterations] => #c0x944834111822843979ace19833d05ca9daf2f655230faec517433e72fe777b
```

And both hashes above open to the same value `123`.

```
lurk-user> (open #c0x944834111822843979ace19833d05ca9daf2f655230faec517433e72fe777b)
[2 iterations] => 123
lurk-user> (open #c0x483cd4ed61cb38d4722743fe470c4c81abf1a568ceae9423864d5acf03739f)
[2 iterations] => 123
```

## Functional commitments

Again, we can commit to *any* Lurk data, including functions.

```
lurk-user> (commit (lambda (x) (+ 7 (* x x))))
[2 iterations] => #c0x6cea5bb3cb63a9bd119d5d00e40807e717e6b98c645ac25aae60ed35255b07
```

The above is a commitment to a function that squares its input then adds seven.
Then we can open it and apply arguments as usual.

```
lurk-user> ((open #c0x6cea5bb3cb63a9bd119d5d00e40807e717e6b98c645ac25aae60ed35255b07) 5)
[8 iterations] => 32
lurk-user> ((open #c0x6cea5bb3cb63a9bd119d5d00e40807e717e6b98c645ac25aae60ed35255b07) 9)
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
[5 iterations] => #c0x4b672917bfcbef3c73e01e50a500f2947d9378f6a1ec19588bc7fd3766f81d
```

Now we can open it, applying it to a function that adds `111` to the secret value that the committed function hides.

```
lurk-user>
((open #c0x4b672917bfcbef3c73e01e50a500f2947d9378f6a1ec19588bc7fd3766f81d)
 (lambda (data) (+ data 111)))
[10 iterations] => 333
```

We coin the term Higher-Order Functional Commitments, and as far as we are aware, Lurk is the first and only extant system enabling this powerful usage in the bare language.
