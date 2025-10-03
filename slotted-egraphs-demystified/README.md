Slotted E-Graphs demystified
============================

In this blog post I attempt (!) to give the simplest explanation of slotted e-graphs that I can come up with.
For that we'll start in Chapter I with some simplifying assumptions, and generalize the system to general slotted e-graphs in Chapters II and III.

# Chapter 1 - From E-Graph to Slotted E-Graph
First, what is an e-graph?
An e-graph stores a bunch of terms and equations among them, by grouping equivalent terms into equivalence classes ("e-classes").
For example, we might represent the term `2*x + 2*y` (with some equations) like this:

```
c0 := 2
c1 := x
c2 := c0 * c1 | c1 + c1
c3 := y
c4 := c0 * c3 | c3 + c3
c5 := c2 + c4
```

Every `c0`, ..., `c5` corresponds to an "e-class", whereas the partial terms on the right (eg. `c0 * c1`) correspond to "e-nodes". [^grammar]
In a slotted e-graph however, every e-class is parameterized by some variables (aka slots).

```
c0 := 2
c1(x) := x
c2(x) := c0 * c1(x) | c1(x) + c1(x)
c3(y) := y
c4(y) := c0 * c3(y) | c3(y) + c3(y)
c5(x, y) := c2(x) + c4(y)
```

It's worth pointing out that variable names chosen in every e-class have no particular meaning, and can be renamed at any point.
We could for example re-define `c2` equivalently as:
```
c2(z) := c0 * c1(z) | c1(z) + c1(z)
```
This is one crucial property of variables: The names you choose never matter!

## Deduplication via Hashcons

In general, E-graphs do not want to store the same term in different classes.
In order to achieve this, there is the "hashcons": the global registry mapping each e-node to the e-class that contains it.
This way we guarantee that any e-node is just contained in at most one e-class.

In Slotted E-Graphs we want an even stronger notion of deduplication:
If two terms are equal up to renaming[^bij] of variables, they should be represented by the same e-class.

The problem now, is that two terms that are equal up to renaming can definitely still hash to different values. Think `hash("x+y") != hash("a+b")`,
so we have to "get rid of the names" before putting our e-nodes into the hashcons. We call this "naming independent representation" a "shape".

To compute the shape of an e-node (or term), we rename all variables to "numeric variables" (`0`, `1`, ...), so for example both `x+y` and `a+b` have the shape `0+1`.[^notation]
In general, we can compute the shape by iterating through the e-node from left to right, and each new variable we encounter will be renamed to `0`, the next one `1`, etc.
(To be clear, if a variable occurs multiple times, all occurrences will be renamed to the same numeric variable.)

If we now populate our hashcons using these shapes, we will notice that both `x` and `y` will result in the shape `0`, which means that we have to merge their e-classes, `c1` and `c3`.[^one-var-eclass]
Or formally: We know that both `c1` and `c3` are able to represent `0`, namely via `c1(0) = 0` and `c3(0) = 0`,
and from that we can infer `c1(0) = c3(0)`.

Note: If the e-classes `c1` and `c3` would have had more variables, the resulting equation (eg. `c7(0, 1) = c9(1, 0)`) would have informed us, which variables from `c1` correspond to which variables from `c3`.
This is crucial as `c7(0, 1) = c9(0, 1)` is a very different statement than `c7(0, 1) = c9(1, 0)`.

So now, we can simplify our slotted e-graph, by replacing all occurrences of `c3(0)` with `c1(0)`, where `0` matches against any variable.

```
c0 := 2
c1(x) := x
c2(x) := c0 * c1(x) | c1(x) + c1(x)
c4(y) := c0 * c1(y) | c1(y) + c1(y)
c5(x, y) := c2(x) + c4(y)

c3(x) := c1(x)
```

And then by again using the hashcons, `c0 * c1(x)` and `c0 * c1(y)` collide at the shape `c0 * c1(0)`. And we similarly merge them.

```
c0 := 2
c1(x) := x
c2(x) := c0 * c1(x) | c1(x) + c1(x)
c5(x, y) := c2(x) + c2(y)

c3(x) := c1(x)
c4(x) := c2(x)
```

## The Unionfind

We have now separated out, the bottom equations. They correspond to the "unionfind" in an e-graph.
Whenever you merge two classes, one will be the "canonical" one (eg. `c1`), and the other one (eg. `c3`) will just point to that canonical class.
It's worth noting that in a slotted e-graph, these unionfind-"pointers" are annotated with renamings.
This has an interesting implication: If you apply path compression in a slotted unionfind, you have to compose the renamings.

# Chapter II - Redundancies (and incidentally also binders)
So, we now have a rough understand how the slotted e-graphs functions.

However, there are a couple of cases that we simply ignored so far.
Let's say we know that `x-x = 0`. This would require us to have an 

# Chapter III - Symmetries
The next (and last) annoying case is that we might learn that `x+y = y+x`.
This is weird, as both sides of the equation are equal up to renaming.
We might try to to "just store them as normal nodes":

```
c0(x) := x
c1(x, y) := c0(x) + c0(y) | c0(y) + c0(x)
```

But this is not enough.
If you know look up the term `a+b`, there's actually two valid outputs. Either `c1(a, b)` or `c1(b, a)`.
Further, a (slotted) e-graph is supposed to "answer" equivalence questions, but if we get the outputs `c1(a, b)` and `c1(b, a)`, we really wouldn't know whether they are supposed to represent the same thing or not.
It depends on whether the class has nodes like `x+y | y+x` or not.

[^bij]: Technically, it would be "equal up to a bijective(!) renaming". As x-y and x-x should not be considered "equal up to renaming".
[^grammar]: If you squint a bit, this looks like a context-free grammar. In general, E-Graphs can be seen as context-free grammars, where non-terminals correspond to e-classes, and production rules correspond to e-nodes. They just have the extra constraint that their non-terminals have no overlap. I'm sure people knew this since the dawn of time, but it's cool and I never see people use that connection somehow.
[^one-var-eclass]: In general, you just have one variable e-class in a slotted e-graph. After all, all variables are equal up to renaming.
[^notation]: Just to be clear, the `2` from `2*x + 2*y` is a *number constant*. And the `0`, `1` from the shape `0+1` are *numeric variables*. They are not the same thing. Sorry for the abuse of notation.
