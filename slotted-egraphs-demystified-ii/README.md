Slotted E-Graphs demystified
============================

- [Chapter I](https://memoryleak47.github.io/slotted-egraphs-demystified-i/).
- [[Chapter II]](https://memoryleak47.github.io/slotted-egraphs-demystified-ii/).
- [Chapter III](https://memoryleak47.github.io/slotted-egraphs-demystified-iii/).

## Chapter II - Redundancies (and incidentally also binders)

Now it's time to lift up the curtain a bit more, and consider what happens when we equate `x-x = Zero`.[^constant]
This equation has an interesting property, the left-hand-side refers to a variable `x`, which does not occur in the right-hand side.

Recall that in any equation, we are always free to bijectively rename all variables: If a property holds for some variables, the property holds for all variables -- no variable is special.
Thus from `x-x = Zero` we can directly infer `y-y = Zero`.
And thus it follows `x-x = y-y`.[^general]

<!-- This new equation `x-x = y-y` is of interest to us, as both sides of the equation are equal up to renaming. -->

If we were to express this situation in a "Chapter I" slotted e-graph, we might attempt this:

```
c0(x) := x
c1(x) := Zero | c0(x) - c0(x)
```

But if we recall the "semantics" `c1(x) = {Zero} ∪ { a - b | a ∈ c0(x), b ∈ c0(x) }`, we notice a problem.
The e-classes `c1(x)` and `c1(y)` overlap in `Zero`, but are not the same.
For example `c1(x)` contains `x-x`, but `c1(y)` does not.
This is not how equivalence classes work! If they overlap, they have to be the same!

This effectively means that the slotted e-graph did not internalize the fact `x-x = y-y` (or `c1(x) = c1(y)`), that we had just proven before.

## Redundant Variables

The property `c1(x) = c1(y)` has an important mathematical characterization, it describes exactly _constant functions_.
We can interpret this as a mathematical hint that the parameter `x` should probably go away.

In order to get rid of the parameter `x`, while still being able to express `x-x`, `y-y`, ..., we require *redundant variables*:

```
c0(x) := x
c1 := Zero | c0(_x_) - c0(_x_)
```

Now, the parameterized e-class `c1` stopped being parameterized, however it now contains a *redundant variable* `_x_`.
The semantics of this is `c1 = {Zero} ∪ { a - b | a ∈ c0(x), b ∈ c0(x), x fresh }`[^fresh], and thus we effectively express `c1 = {Zero, x-x, y-y, ...}` as desired!

Notice that this semantics is exactly the union of all these e-classes `c1(x)`, `c1(y)`, ... that were overlapping but not equivalent in the previously attempted semantics.
Thus, in typical e-graph fashion, we merged a couple of overlapping e-classes into one.

## Binders

When seeing the equation `x-x = y-y`, someone might have already figured out how this could relate to binders.
Binders have a property called *alpha-equivalence*; for example `λx. x = λy. y` is a consequence of alpha-equivalence.
It basically states that you are free to rename any bound variables in a term, and you will obtain an equivalent term.

<!-- In short: `t = t[x := y]` if `x` is a bound variable, and `y` is fresh. -->

So, the trick to encode binders in slotted e-graphs is to just have a normal `λx. _` e-node type.[^lambda]
And whenever some e-class `c(x, y, z)` contains an e-node `λx. _`,
you derive `c(x, y, z) = c(x', y, z)` and let the redundancy system take it from there.

## Rebuilding

As noted in "Chapter I", learning new equations and simplifying accordingly can sometimes (by congruence) yield new equations.
A similar thing happens with redundant variables.
If you know that `c1(x) = x-x` gets simplified to `c1` without a parameter,
then all "parent" e-classes (like `c4(x, y, z) = c1(x) + c2(y, z)`) may lose the dependency on this parameter aswell (like `c4(y, z) = c1 + c2(y, z)`).

In an extreme case, when equating `x=y`, then the unique variable e-class `c0(x) := x` gets a redundant slot `c0 := _x_`,
and all other e-classes lose all their slots as a consequence of this. Then the slotted e-graph degenerates to a conventional e-graph.

## Freshness and bijectiveness

In general, the equational reasoning underlying slotted e-graphs is very related to nominal techniques.
One consequence of that, which we mentioned along the way, was that all our renamings are "bijective".
Implying that we never rename two originally different variables to the same new variable.[^reason]

To make an example: `x-y` and `x-x` are entirely unrelated e-nodes in the slotted setting.
There is no bijective renaming one can apply to go from one to the other, in either direction.

This is of particular relevance for redundant slots, as their semantics is explicitly based on freshness.
For that, let's consider the example `λx. λy. x+y`:

```
c0(x) := x
c1(x,y) := x+y
c2(x) := λ_y_. c1(x, _y_)
c3 := λ_x_. c2(_x_)
```

When we look at the semantics of `c2(x)`, we obtain

- `c2(x) = { λy. a | a ∈ c1(x, y), y fresh }`

This "freshness" constraint prevents `c2(x)` from containing `λx. x+x`. Great! Crisis averted.

I want to point out that "x fresh" in this context does not mean "globally fresh",
but instead: x is distinct from all other variables explicitly mentioned in this equation.


[^constant]: I write "Zero" to prevent being ambiguous with the numeric variables `0, 1, 2` that we use in shapes.
[^general]: This works generally. If you have an equation `t1 = t2`, where `x` comes up in `t1`, but not in `t2`, you can always derive `t1[x := y] = t2` and thus `t1 = t1[x := y]` (assuming `y` fresh).
[^lambda]: One thing that is a bit special about the λ-node is that, next to the "variable" e-node, it's a node that takes not only subterms, but also a variable (the bound one) directly. If you don't do this, you get weird terms like `λ(x+0). _` when `x = x+0` (but there's also other ways to fix that).
[^reason]: You might also have wondered why no e-classes ever contain nodes like `c2(x, x)`: This would correspond to a non-bijective renaming, and is thus forbidden.
[^fresh]: What exactly "fresh" means will be explained in [Freshness](#freshness).
