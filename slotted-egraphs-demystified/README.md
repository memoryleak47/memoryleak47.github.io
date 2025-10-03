Slotted E-Graphs demystified
============================

In this blog post I attempt to give the simplest explanation of slotted e-graphs that I can come up with.
For that we'll start in Chapter I with a simplified version, and generalize it to general slotted e-graphs in Chapters II and III.

# Chapter I - From E-Graph to Slotted E-Graph
First, what is an e-graph?
An e-graph stores a bunch of terms and equations among them, by grouping equivalent terms into equivalence classes ("e-classes").
For example, we might represent the equal terms `-(x-y) = (y-x)` as follows:

```
c0 := x
c1 := y
c2 := c0 - c1
c3 := - c2 | c1 - c0
```

Every `c0, ..., c3` corresponds to an "e-class", whereas the terms on the right (eg. `c0 - c1`) correspond to "e-nodes". [^grammar]

If we convert this conventional e-graph to a slotted e-graph, every e-class will be parameterized by the variables (aka slots) that are used in this e-class:

```
c0(x) := x
c1(y) := y
c2(x, y) := c0(x) - c1(y)
c3(x, y) := - c2(x, y) | c1(y) - c0(x)
```

One can visualize a slotted e-class (eg. `c3`) as a function, that takes in some names (`x, y`) and yields a set of terms `- c2(x, y)` unioned with `c1(y) - c0(x)`.
These e-class functions recursively call each other to build up all the terms represented by an e-class.

As slotted e-classes can be seen as functions, the variable names (i.e. function parameters) chosen in every e-class have no particular meaning, and can be renamed at any point.
We could for example re-define `c2` equivalently as `c2(f, d) := c0(f) - c1(d)` if we wanted to.
This is one crucial property of variables: The names you choose do not matter!

Further, it's worth noting that each id `c0` spans infinitely many different e-classes `c0(x)`, `c0(y)`, ...,
that all represent disjoint sets of terms.
[TODO: clarify e-class vs. applied e-class? What do I call "e-class"?]

### Deduplication via Hashcons and Shapes

In general, E-graphs want to prevent storing the same e-node in multiple e-classes.
In order to achieve this, there is the "hashcons": the global registry mapping each e-node to the e-class that contains it.
This way we guarantee that any e-node is contained in at most one e-class.

In Slotted E-Graphs we want an even stronger notion of deduplication:
If two terms are equal up to renaming[^bij] of variables, they should be represented by the same e-class.

The problem now, is that two terms that are equal up to renaming can definitely still hash to different values. Think `hash("x+y") != hash("a+b")`,
so we have to "get rid of the names" before putting our e-nodes into the hashcons. We call this "naming independent representation" a "shape".

To compute the shape of an e-node (or term), we rename all variables to "numeric variables" (`0`, `1`, ...), so for example both `x+y` and `a+b` have the shape `0+1`.
In general, we can compute the shape by iterating through the e-node from left to right, and each new variable we encounter will be renamed to `0`, the next one `1`, etc.
(To be clear, if a variable occurs multiple times, all occurrences will be renamed to the same numeric variable.)

### Continuing the example

If we now come back to the example from before, and populate the hashcons in the way we have just described,
we will notice that both `x` and `y` will result in the shape `0`, yielding a "hashcons-collision".
Our hashcons-invariant forces us to map `hashcons[0]` to both `c0` and `c1`, which means that we have to merge these e-classes.[^one-var-eclass]

How do we know that these e-classes should be merged? We know that both `c0` and `c1` are able to represent `0`, namely via `c0(0) = 0` and `c1(0) = 0`,
and from that we can infer `c0(0) = c1(0)`.

So now we can simplify our slotted e-graph, by replacing all occurrences of `c1(0)` with `c0(0)`, where `0` matches against any variable:

```
c0(x) := x
c2(x, y) := c0(x) - c0(y)
c3(x, y) := - c2(x, y) | c0(y) - c0(x)

c1(x) := c0(x)
```

### "Re-ordering" equations

This simplification enables another one:
The e-classes `c2` and `c3` now each contain an e-node with shape `c0(0) - c0(1)`, that we will detect when populating the hashcons.
From these e-nodes we infer the equation `c2(0,1) = c0(0) - c0(1) = c3(1,0)`.[^shape-compute]
Note that this equation `c2(0,1) = c3(1,0)` is a very different equation than `c2(0,1) = c3(0,1)`; as we have to flip the variables!

We then choose to replace `c3(1,0)` with `c2(0,1)` in the slotted e-graph, where again `0, 1` can match against any variables:

```
c0(x) := x
c2(x, y) := - c2(y, x) | c0(x) - c0(y)

c1(x) := c0(x)
c3(x, y) := c2(y, x)
```

### The Unionfind

Whenever you merge two classes, one will be the "canonical e-class" (in this case `c2`), and the other e-class (`c3`) will just point to that canonical e-class.
This "pointer" `c3(x, y) := c2(y, x)` will be stored in a unionfind datastructure.
In this case just with an extra layer of parameters to express in which way the slots get changed.

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
[^shape-compute]: We obtain for example `c2(0,1)` as follows: `c2(x, y)` contains the e-node `c0(x) - c1(y)`, and during shape computation we remember the renaming that we need to apply to obtain the shape `c0(0) - c1(1)`. In this case `[x := 0, y := 1]` maps the e-node `c0(x) - c1(y)` to its shape `c0(0) - c1(1)`. Applying this renaming on `c2(x,y)` yields the final `c2(0,1)`.
