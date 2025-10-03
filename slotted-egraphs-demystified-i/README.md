Slotted E-Graphs demystified
============================

In this blog post I attempt to give a more concise and simple explanation of slotted e-graphs, than the one from our paper.
For that we'll start in Chapter I with a simplified version of slotted e-graphs, and generalize it to general slotted e-graphs in Chapters II and III.

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

If we convert this conventional e-graph to a slotted e-graph,
every e-class will be parameterized by the variables (aka slots) that it contains:

```
c0(x) := x
c1(y) := y
c2(x, y) := c0(x) - c1(y)
c3(x, y) := - c2(x, y) | c1(y) - c0(x)
```

One can visualize a parameterized e-class (eg. `c3`) as a function, that takes in variable names (`x, y`) and yields a set of terms:

- `c3(x, y) = {-a | a ∈ c2(x, y) } ∪ { a - b | a ∈ c1(y), b ∈ c0(x)}`

These parameterized e-class functions recursively call each other to build up all the terms they represent.[^termination]

<!-- It's worth pointing out that a parameterized e-class (`c0`) spans infinitely many disjoint equivalence classes: -->
<!-- `c0(a)`, `c0(b)`, ...; one e-class per "application" of the parameterized e-class.[^groups] -->

As parameterized e-classes can be seen as functions, the variable names (i.e. function parameters) chosen in every e-class have no particular meaning, and can be renamed at any point.
We could for example re-define `c2` equivalently as `c2(f, d) := c0(f) - c1(d)` if we wanted to.
This is one crucial property of variables: The names you choose do not matter!

### Deduplication via Hashcons and Shapes

As each term should be represented only in a single e-class, e-graphs must detect when different e-classes share a common e-node.
In order to achieve this they use the "hashcons", which is supposed to map each e-node to the unique e-class that contains it.

In Slotted E-Graphs we want a stronger notion of deduplication:
If two e-nodes (or terms) are equal up to renaming[^bij] of variables, they should be represented by the same parameterized e-class.

The problem with the hashcons is that two e-nodes that are equal up to renaming can definitely still hash to different values. Think `hash("c7(x) + c2(y)") != hash("c7(a) + c2(b))")`,
so we require a "name-independent representation" of e-nodes (called its *shape*) that we can use for the hashcons.

To compute the shape of an e-node, we rename all variables to "numeric variables" (`0`, `1`, ...) based on the order of their first occurrence.
For example the e-node `c3(x, y, z) + c4(w, x)` would have the shape `c3(0, 1, 2) + c4(3, 0)`.
In other words, the shape of an e-node is the lexicographically smallest e-node that is equal up to renaming to it. (Assuming the lexicographical ordering `0 < 1 < 2 < ...`)

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

c1(0) := c0(0)
```

We remember how we eliminated `c1` using this final equation `c1(0) := c0(0)`. We will clarify this more in the "Unionfind" subsection.

### Non-trivial equations

This simplification enables another one:
The e-classes `c2` and `c3` now each contain an e-node with shape `c0(0) - c0(1)`, that we will detect when populating the hashcons.
From these e-nodes we infer the equation `c2(0,1) = c0(0) - c0(1) = c3(1,0)`.
This equation is a bit more interesting than the one we had before. We have equated the e-classes `c2` and `c3`, but there's a *literal* twist:
The variable `x` in `c2`, corresponds to the variable `y` in `c3` and vice versa. Hence we cannot equate `c2(0,1) = c3(0,1)`, but we have to be careful to respect this renaming.

Using this new equation, We can now replace `c3(1,0)` with `c2(0,1)`, where again `0, 1` can match against any variables:

```
c0(x) := x
c2(x, y) := - c2(y, x) | c0(x) - c0(y)

c1(0) := c0(0)
c3(0, 1) := c2(1, 0)
```

### Unionfind

Whenever you merge two classes, one will be the "canonical e-class" (in this case `c2`), and the other e-class (`c3`) will just point to that canonical e-class.
This "pointer" `c3(0, 1) := c2(1, 0)` will be stored in a unionfind datastructure, it is expressed as `unionfind[c3] = c2(1, 0)`.[^impl]
When applying path compression to the unionfind, one has remember to compose these re-orderings;
just like `c2(x, y) := c1(y, x)` and `c1(x, y) := c0(y, x)` implies `c2(x, y) = c0(x, y)`.

[^bij]: Technically, it would be "equal up to a bijective(!) renaming". As x-y and x-x should not be considered "equal up to renaming".
[^grammar]: If you squint a bit, this looks like a context-free grammar. In general, E-Graphs can be seen as context-free grammars, where non-terminals correspond to e-classes, and production rules correspond to e-nodes. They just have the extra constraint that their non-terminals have no overlap. I'm sure people knew this since the dawn of time, but it's cool and I never see people use that connection somehow.
[^one-var-eclass]: In general, you just have one variable e-class in a slotted e-graph. After all, all variables are equal up to renaming.
[^groups]: This is using simplified assumptions: In chapter III, we will see that different applications of a p-class will not always yield different e-classes.
[^impl]: In the current implementation, we actually store `unionfind[c3] = (c2, [x := y, y := x])`, as it uses canonical names (`x, y`) instead of canonical positions (`0, 1`). But that's a matter of taste.
[^termination]: Technically, for recursive e-classes these functions would not terminate, but I hope it's clear what I mean. Think of these sets as the smallest fixed-points under these equations.
