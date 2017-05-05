Deprecate Tuple Shuffles
========================

-   Proposal: [SE-NNNN](NNNN-filename.md)
-   Authors: [Robert Widmann](https://github.com/codafi) 
-   Review Manager: TBD
-   Status: **Awaiting review**

Introduction
------------

This proposal seeks the deprecation of a little-known feature of Swift called
a "Tuple Shuffle".

Motivation
----------

A tuple-shuffle is an undocumented feature of Swift in which one can re-order
the indices of a tuple by writing a pattern that describes a permutation in a
syntax reminiscent of adding type-annotations to a parameter list:

```swift
let a = (x: 1, y: 2)
var b: (y: Int, x: Int)
b = a
```

It can be used to simultaneously destructure and reorder a tuple:

```swift
let tuple = (first: 0, second: (x: 1, y: 2))
let (second: (x: b, y: c), first: a) = tuple
```

It can also be used to map parameter labels out of order in a call expression:

```swift
func foo(_ : (x : Int, y : Int)) {}
foo((y: 5, x: 10)) // Valid
```

Note that a tuple shuffle is distinct from a re-assignment through a tuple pattern.
For example, this series of statements will continue to function as before:

```swift
var x = 5
var y = 10
var z = 15
(z, y, x) = (x, z, y)
```

Their inclusion in the language complicates every part of the compiler stack, uses
a [syntax that can be confused for type annotations](https://twitter.com/CodaFi_/status/860246169854894081),
contradicts the goals of earlier SE's (see [SE-0060](https://github.com/apple/swift-evolution/blob/9cf2685293108ea3efcbebb7ee6a8618b83d4a90/proposals/0060-defaulted-parameter-order.md)), 
and makes non-sensical patterns possible in surprising places. 

Take switch-statements, for example:


```swift
switch ((0, 0), 0){ 
case (_ : let (y, z), _ : let s): () // We are forbidden from giving these patterns names other than "_" 
default: () 
}
```
  
This proposal seeks to deprecate them in Swift 3 compatibility mode and enforce that 
deprecation as a hard error in Swift 4 to facilitate their eventual removal from the
language.


Proposed solution
-----------------

Construction of Tuple Shuffle Expressions will become a warning in Swift 3 compatibility mode
and will be a hard-error in Swift 4.

Detailed design
---------------

In addition to the necessary diagnostics, the grammar will be ammended to 
simplify the following productions:

```diff
tuple-pattern → (tuple-pattern-element-list <opt>)
tuple-pattern-element-list → tuple-pattern-element | tuple-pattern-element , tuple-pattern-element-list
- tuple-pattern-element → pattern | identifier:pattern
+ tuple-pattern-element → pattern
```

Impact on Existing Code
-----------------------

Because very little code is intentionally using Tuple Shuffles, impact on existing code will be
negligible but not non-zero.  

Alternatives considered
-----------------------

Continue to keep the architecture in place to facilitate this feature.

