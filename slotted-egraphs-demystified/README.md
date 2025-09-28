Slotted E-Graphs demystified
============================

[This is work in progress. Probably not gonna be useful.]

In this blog post I attempt (!) to give the simplest explanation of slotted e-graphs that I can come up with.
For that we'll start in Chapter 1 with some simplifying assumptions, and generalize the system to general slotted e-graphs in Chapters II and III.
This blog post will focus more on conveying the idea, than about being precise in-terms of data structures.

# Chapter I - The basic idea of slotted e-graphs

General E-Graphs suffer from the problem that they don't know what variables are.
Variables are placeholders: A variable "x" typically signifies a placeholder, where we don't want to decide yet what will take its place in the future.
One crucial property of variables is that the actual name chosen for them is irrelevant.
We humans, just *see* that the following two python programs are doing the same thing:

```
c = 0
for i in range(10):
    c += i
return c
```

and

```
d = 0
for j in range(10):
    d += j
return d
```

But if you wanted to encode them into an e-graph, the e-graph will consider them to be very much different, as the e-graph does not "abstract" the variable names "i,j,c,d" away.
For the e-graph a variable "i" is just a symbol[^1], like the mathematical constant "pi"; and thus a different symbol from the variable "j".

An e-graph groups terms into different equivalence classes (e-classes).
We will write `[t]` for the equivalence class representing the term `t`.

However, a slotted e-graph has a different notion of equivalence classes, namely "equivalence up to renaming".
We noted earlier, that "x+y" and "a+b" are conceptually really similar terms. If we choose to rename "x" to "a", and "y" to "b"

# Chapter II - Redundant Slots
# Chapter III - Symmetries

[^1]: This assume that we encode the variables as "symbols" into the e-graph. If we use a de-bruijn encoding, we get into similar problems. I go more into this in the PLDI 2025 talk.
