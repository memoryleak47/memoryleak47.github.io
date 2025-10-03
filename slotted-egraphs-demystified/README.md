Slotted E-Graphs demystified
============================

In this blog post I attempt (!) to give the simplest explanation of slotted e-graphs that I can come up with.
For that we'll start in Chapter I with some simplifying assumptions, and generalize the system to general slotted e-graphs in Chapters II and III.

Contrary to how it is described in the PLDI paper, we will use natural numbers `0, 1, 2, 3, ...` to express variables (aka slots).
This can make stuff a bit easier, but might be unintuitive at first.[TODO: clarify why]

# Chapter 1 - From E-Graph to Slotted E-Graph
First, what is an e-graph?
An e-graph stores a bunch of terms and equations among them, by grouping equivalent terms into equivalence classes ("e-classes").
For example, we might represent the equation `-(0-1) = 1-0` like this (again, `0` and `1` are variables here):

```
a := 0
b := 1
c := a - b
d := - c = b - a
```

Every `a, ..., d` corresponds to an "e-class", whereas the terms on the right (eg. `a - b`) correspond to "e-nodes".[^grammar]

If we want to convert this e-graph to a slotted e-graph,
In a slotted e-graph however, every e-class is parameterized by some variables (aka slots).
So the previous e-graph becomes this slotted e-graph:

```
a(0) := 0
b(0) := 0
c(0, 1) := a(0) - b(1)
d(0, 1) := - c(0, 1) = b(1) - a(0)
```

## Deduplication via Hashcons

In general, E-graphs do not want to store the same term in different classes.
In order to achieve this, there is the "hashcons": the global registry mapping each e-node to the e-class that contains it.
This way we guarantee that any e-node is just contained in at most one e-class.

When we build up the hashcons, we will note that the e-node `0` comes up both in `a(0)` and in `b(0)`.
Thus `a(0) = b(0)`, and we can choose to replace all `b(0)` with `a(0)`, while `0` is allowed to match against any variable that comes up.
After doing this, we obtain:

```
a(0) := 0
c(0, 1) := a(0) - a(1)
d(0, 1) := - c(0, 1) = a(1) - a(0)

b(0) := a(0)
```

Note that this already guarantees that we have exactly one "variable e-class", as all variables are generally "equal to up to renaming".
We keep the rule `b(0) := a(0)` that we just applied, at the bottom of our list of equations.
This corresponds to a "unionfind" entry, that normalizes `b(0)` to `a(0)`. But more on that later.

## A slotted hashcons

In Slotted E-Graphs we want a stronger notion of deduplication than conventional e-graphs:
If two terms are equal up to renaming[^bij] of variables (or for numeric variables, "re-ordering"), they should be represented by the same e-class.

As we can see in the example, this didn't work out yet: adding `x-y` to the slotted e-graphs produced the e-class `c(x, y)`, and adding `y-x` produced `d(y, x)`.

To fix this problem, we have to "canonicalize" the e-nodes that we put into the hashcons, by reordering the variables from small to big.
Then both `a(0) - a(1)` and `a(1) - a(0)` will become `a(0) - a(1)`. We call this "canonicalized" e-node, the "shape" of the e-node.

If we now populate our hashcons using these shapes, we will find a collision, and notice that `c(0, 1) = a(0) - a(1) = d(1, 0)`.
It's crucial that `c(0, 1) = d(1, 0)` is an entirely different equation from `c(0, 1) = d(0, 1)`.
In general, there are multiple ways to equate the same e-classes, depending on which variables correspond to which.
Using this new equation, we can again replace all `d(1, 0)` with `c(0, 1)`.

```
a(0) := 0
c(0, 1) := a(0) - a(1) = - c(1, 0)

b(0) := a(0)
d(0, 1) := c(1, 0)
```

# Chapter II - Redundancies (and incidentally also binders)
So, we now have a rough understand how the slotted e-graphs functions.

However, there are a couple of cases that we simply ignored so far.
Let's say we know that `x-x = 0`. This would require us to have an 

# Chapter III - Symmetries
The next (and last) annoying case is that we might learn that `x+y = y+x`.
This is weird, as both sides of the equation are equal up to renaming.
We might try to to "just store them as normal nodes":

```
TODO
```

But this is not enough.
If you know look up the term `a+b`, there's actually two valid outputs. Either `c1(a, b)` or `c1(b, a)`.
Further, a (slotted) e-graph is supposed to "answer" equivalence questions, but if we get the outputs `c1(a, b)` and `c1(b, a)`, we really wouldn't know whether they are supposed to represent the same thing or not.
It depends on whether the class has nodes like `x+y | y+x` or not.

[^bij]: Technically, it would be "equal up to a bijective(!) renaming". As x-y and x-x should not be considered "equal up to renaming".
[^grammar]: If you squint a bit (or replace the `=` with `::=` and `|`), this e-graph is a context-free grammar. Non-terminals correspond to e-classes, and production rules correspond to e-nodes. I'm sure people knew this connection since the dawn of time, but it's cool and I never see people use that connection somehow.
