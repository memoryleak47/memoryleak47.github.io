Slotted E-Graphs demystified
============================

- [Chapter I](https://memoryleak47.github.io/slotted-egraphs-demystified-i/).
- [Chapter II](https://memoryleak47.github.io/slotted-egraphs-demystified-ii/).
- [[Chapter III]](https://memoryleak47.github.io/slotted-egraphs-demystified-iii/).

# This is not written yet! WIP

## Chapter III - Symmetries
Now, we have seen how slotted e-graphs address the case where the sides of an equation used different variables;
the final problem to address are symmetries.

And for that, consider the equation `x+y = y+x`.
This equation is interesting as both sides sides of the equation are equal up to renaming (similarly to `x-x = y-y` btw),
and thus represent an inherent property of the e-class that represents the addition of two different variables, and only that e-class.

If we want to express this situation in a "Chapter II" slotted e-graph, we might attempt this:
```
c0(x) := x
c1(x, y) := c0(x) + c0(y) | c0(y) + c0(x)
```

But there's an issue: For example the term `x + y` is now represented by two applied e-classes: `c1(x, y)` and `c1(y, x)`.
This is detrimental, as we require a canonical representative for any term, in order to do efficient equivalence checking.

### Group Canonicalization

In this case it might be reasonably simple, say "just pick the lexicographically smallest one as representative": `c1(x, y)`.

But how do you compute this lexicographically smallest one in general?

Let's say we have a bunch of equations:
- `c1(x, y, z, w) = c1(y, z, w, x)`
- `c1(x, y, z, w) = c1(y, x, z, w)`?
- Now what's the normal form of `c1(z, w, y, x)`?

At the risk of losing some readers, let me rephrase this as:
- `c1 = c1[x := y, y := z, z := w, w := x]`
- `c1 = c1[x := y, y := x, z := z, w := w]`
- Now what's the normal form of `c1(z, w, y, x)`?

These mappings `[x := y, ...]` are "renaming" functions, and they compose like typical functions do: `c[x := y][y := z] = c[x := z]`.

There's a known algorithm for this, one can first use the Schreier-Sims algorithm to build a stabilizer chain;
and then use it quickly determine the smallest lexicographical element.

### Rebuilding

Similarly to redundancies, symmetries can also propagate using congruence.
Think about `x+y = y+x` implying `f(x+y) = f(y+x)`.

### Shape computation

<!-- or weak shapes? -->
Recall that a "shape" of an e-node is the lexicographically minimal e-node that is equivalent[^equiv] to it.

Now, if we have groups, this becomes a bit more tricky:
Computing the shape of `c1(x, y) + c2(y, x)` might actually be `c1(0, 1) + c2(0, 1)`, if `c1(x, y) = c1(y, x)` holds.

[^equiv]: equivalent in this case means 1) up to renaming, and 2) up to symmetry equivalences, like `c1(x, y) = c1(y, x)`.
