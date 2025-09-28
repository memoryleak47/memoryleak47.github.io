Slotted E-Graphs demystified
============================

[This is work in progress. Probably not gonna be useful.]

In this blog post I attempt (!) to give the simplest explanation of slotted e-graphs that I can come up with.
For that we'll start in Chapter 1 with some simplifying assumptions, and generalize the system to general slotted e-graphs in Chapters II and III.
This blog post will focus more on conveying the idea, than about being precise in-terms of data structures.

# Chapter 0 - Basics.

## What is an e-graph

An E-Graph efficiently stores a bunch of terms, grouped into equivalence classes.
With brutal simplifications[^egraph-simp], you can think of an e-graph like this:
```rust
type Id = usize; // e-class id.

struct EGraph {
    classes: Map<Id, Class>,
}

// An equivalence class of terms.
struct Class {
    nodes: Vec<Node>,
}

struct Node {
    operator: String,
    children: Vec<Id>,
}
```

## What is a variable?

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
For the e-graph a variable "i" is just a symbol[^symbol], like the mathematical constant "pi"; and thus a different symbol from the variable "j".

An e-graph groups terms into different equivalence classes (e-classes).
We will write `[t]` for the equivalence class representing the term `t`.

However, a slotted e-graph has a different notion of equivalence classes, namely "equivalence up to renaming".
Similarly to the python example above, the terms "x+y" and "a+b" are conceptually really similar terms (i.e. they are equal up to renaming of variables).
If we choose to rename "x" to "a", and "y" to "b" (or vice versa), we end up making these terms the same.

# Chapter I - The basic idea of slotted e-graphs

As slotted e-graphs want to abstract over names, so that "x+y" and "a+b" can be represented using the same general e-class,
we "decompose" the variable names from the structure of the terms:
We decompose "x+y" into "0+1" where "0" maps to "x" and "1" maps to "y". Similarly,
we decompose "a+b" into "0+1" where "0" maps to "a" and "1" maps to "b".

This way, both "x+y" and "a+b" have the same *shape* "0+1", but they choose to map "0" and "1" to other variable names.
Now, in a slotted e-graph we would store this shape "0+1" in some e-class, say "c".
and then reference to "a+b" by writing "c[0 := a, 1 := b]", or simply just "c[a,b]".
Similarly we reference "x+y" by writing "c[x, y]".

These *things* "c[a,b]" are what we call an "applied id", or "renamed id". It's basically a slotted e-class, applied with a bunch of names for each "name argument" that the e-class requires.

```rust
type Id = usize; // e-class id.
type Slot = String; // a variable name.

// think "c[x,y]", where "c" is the id, and "[x, y]" are the children.
struct AppliedId {
    id: Id,
    children: Vec<Slot>,
}

struct SlottedEGraph {
    classes: Map<Id, Class>,
}

struct Class {
    slots: Vec<Slot>, // The slots that this class uses.
    nodes: Vec<Node>,
}

enum Node {
    Op(/*operator*/ String, /*children*/ Vec<AppliedId>),
    Variable(String),
}
```

# Chapter II - Redundant Slots
# Chapter III - Symmetries

[^egraph-simp]: We ignore the "unionfind" data structure, as it's just an optimization. Instead of using a unionfind to update stuff, you can always iterate through the whole e-graph and replace one Id with the other. (This doesn't work if the user got access to an e-class Id that we remove, but whatever.). We ignore the hashcons, as you can also just search through all classes. It's slow, but equivalent.
[^symbol]: This assume that we encode the variables as "symbols" into the e-graph. If we use a de-bruijn encoding, we get into similar problems. I go more into this in the PLDI 2025 talk.
