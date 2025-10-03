Slotted E-Graphs demystified
============================

In this blog post I attempt (!) to give the simplest explanation of slotted e-graphs that I can come up with.
For that we'll start in Chapter I with some simplifying assumptions, and generalize the system to general slotted e-graphs in Chapters II and III.
This blog post will focus more on conveying the idea, than about being precise in-terms of data structures.

# Chapter 1 - From E-Graph to Slotted E-Graph
First, what is an e-graph?
An e-graph stores a bunch of terms and equations among them, by grouping equivalent terms into equivalence classes ("e-classes").
For example, if we know that "2\*x = x+x" and "2\*y = y+y" and we want to represent the term "2\*x + 2\*y" it will result in the following e-graph:

c0 := x
c1 := 2\*c0 | c0 + c0
c2 := y
c3 := 2\*c2 | c2 + c2
c4 := c1 + c3
[^grammar]

Every c0, ..., c4 corresponds to an "e-class", whereas the partial terms on the right x, 2\*c0, ... correspond to e-nodes.

In a slotted e-graph, this remains the same -- however classes get parameterized by their variables (slots):
So the same e-graph would now be:

c0(x) := x
c1(x) := 2\*c0(x) | c0(x) + c0(x)
c2(y) := y
c3(y) := 2\*c2(y) | c2(y) + c2(y)
c4(x, y) := c1(x) + c3(y)

Now, e-graphs do not want to store the same term in different classes.
In order to achieve this, there is the "hashcons", the global registry expressing where a "node" is contained.
This way we guarantee that any node is just contained in at most one class.

In Slotted E-Graphs we want an even stronger notion of deduplication:
If two terms are equal up to renaming[^bij] of variables, they should be represented by the same e-class.

The problem now, is that two terms that are equal up to renaming can definitely still hash to different values. Think hash("x+y") != hash("a+b"), so we can't simply take the hashcons as before.
In order to solve this issue, we are required to normalize our names in a sense that is compatible to hashing.

For this, we rename all slots to numbers: we identify "0" with the left-most-occurring variable, then 1 next, etc.
In this sense, both "x+y" and "a+b" would get the output "0+1".[^shape]

If we now populate our hashcons, we will notice that both "x" and "y" will result in the shape "0".
And thus we equate c2 = c0.

c0(x) := x
c1(x) := 2\*c0(x) | c0(x) + c0(x)
c3(y) := 2\*c0(y) | c0(y) + c0(y)
c4(x, y) := c1(x) + c3(y)



[^bij]: Technically, it would be "equal up to a bijective(!) renaming". As x-y and x-x should not be considered "equal up to renaming".
[^grammar]: If you squint a bit, this looks like a context-free grammar. In general, E-Graphs can be seen as context free grammars, where non-terminals correspond to e-classes, and production rules correspond to e-nodes. They just have the extra constraint that their non-terminals have no overlap. I'm sure people knew this since the dawn of time, but it's cool and I never see people use that connection somehow.
[^shape]: We call this a "shape" in the paper.
