# Introduction

### Provable, verifiable and Turing-complete

Lurk is a Turing-complete programming language whose execution can be proved and verified using zk-SNARKs.
So while most Turing-complete programming languages can't generate proofs of their correct execution and most zk-SNARK authoring languages aren't Turing-complete, Lurk offers the best of both worlds.

### Lurk is a Lisp

At a higher level, Lurk is designed as a statically scoped dialect of Lisp, influenced by Scheme and Common Lisp.
However, the consequences of Lurk being a Lisp aren't limited to its frontend.
It facilitates a direct relationship between Lurk expressions and Lurk data.
In fact, Lurk expressions *are* Lurk data.
We hope to clarify this concept throughout the book.

### Lurk's computational model

As its computational model, Lurk implements a CEK machine.
So, more precisely, a proof of execution for a Lurk program is a proof that Lurk's CEK machine goes from state *a* to state *b*, where *a* encodes the computation that we want to unroll and *b* qualifies a final state, containing the result of the computation.

### No compilation to circuit

Lurk is an *interpreted* language.
Program authors aren't required to compile their programs to circuits before being able to generate proofs.
Instead, Lurk implements an *universal circuit* that can prove the transition from a CEK state to the next.
These proofs are then *folded* in order to create a single succint and cryptographic proof.

This design outlines a striking characteristic of Lurk.
In a Lurk program that implements a logical fork with an `if`, the prover only needs to pay for the cost of the path that was actually taken.
Whereas the typical circuit compilation approach would have to create a circuit that contemplates the constraints for both paths, always.

### Disclaimer

Lurk is currently in Alpha.
Code that runs in the Lurk Alpha release is expected to also run in Lurk Beta, and eventually Lurk 1.0.
However, some low-level data representations are anticipated to change, and we will be refactoring the circuit implementation to increase auditability and further our confidence in Lurk's cryptographic security.
Also note that since Lurk inherits some security properties from the underlying proving system.
Those who would rely on Lurk should investigate the security and status of Nova itself.

We encourage early adopters to begin writing real applications taking advantage of Lurk so you can begin to familiarize yourself with the programming model.
Likewise, we welcome your feedback -- which will help ensure ongoing development meets user need.

Note that Groth16 support is only partial, since no trusted setup has been run. If you wish to use Lurk with Groth16, a trusted setup will be required; and we would not recommend undertaking this until the 1.0 circuit has been released.

### About this book

This book takes a "learn by examples" path, providing just enough information at each step and leaving more fine-grained details for later.

The "Language specification" chapter can be used as a precise reference for experienced Lurk users.
