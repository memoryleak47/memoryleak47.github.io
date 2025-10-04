# This is not written yet! WIP

# Chapter II - Redundancies (and incidentally also binders)

Now it's time to lift up the curtain a bit more, and consider what happens when we equate `x-x = Zero`.[^constant]
This equation has an interesting property, the left-hand-side refers to a variable `x`, which does not occur in the right-hand side.

Recall that in any equation, we are always free to bijectively rename all variables: If a property holds for some variables, the property holds for all variables -- no variable is special.
Thus from `x-x = Zero` we can directly infer `y-y = Zero`.
And thus it follows `x-x = y-y`.[^general]

<!-- This new equation `x-x = y-y` is of interest to us, as both sides of the equation are equal up to renaming. -->

If we were to express this situation in a slotted e-graph, we might start here:

```
c0(x) := x
c1(x) := Zero | c0(x) - c0(x)
```

But if we recall the "semantics" `c1(x) = {Zero} ∪ { a - b | a ∈ c0(x), b ∈ c0(x) }`, we notice a problem.
The e-classes `c1(x)` and `c1(y)` overlap in `Zero`, but are not the same.
For example `c1(x)` contains `x-x`, but `c1(y)` does not.
This is not how equivalence classes work. If they overlap, they have to be the same!
This effectively means that the slotted e-graph did not internalize the fact `x-x = y-y` (or `c1(x) = c1(y)`), that we had just proven before.

In order to address this, we require *redundant variables*:

```
c0(x) := x
c1 := Zero | c0(_x_) - c0(_x_)
```

Now, the parameterized e-class `c1` stopped being parameterized, however it now contains a *redundant variable* `_x_`.
The semantics of this is `c1 = {Zero} ∪ { a - b | a ∈ c0(x), b ∈ c0(x), ∀x }`.
<!-- Hm... the x is technically not allowed to overlap other things, as we will see. -->

## Binders

When considering the equation `x-x = y-y`, someone might have guessed how this could relate to binders.
Binders have a property called *alpha-equivalence*; for example `λx. x = λy. y` is a consequence of alpha-equivalence.
It basically states that you are free to rename any bound variables in a term, and you will obtain an equivalent term.

In short: `t = t[x := y]` if `x` is a bound variable, and `y` is fresh.


[^constant]: I write "Zero" to prevent being ambiguous with the numeric variables `0, 1, 2` that we use in shapes.
[^general]: This works generally. If you have an equation `t1 = t2`, where `x` comes up in `t1`, but not in `t2`, you can always derive `t1[x := y] = t2` and thus `t1 = t1[x := y]` (assuming `y` fresh).
