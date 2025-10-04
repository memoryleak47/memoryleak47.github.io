# This is not written yet! WIP

# Chapter II - Redundancies (and incidentally also binders)

Now it's time to lift up the curtain a bit more, and consider what happens when we equate `x-x = Zero`.[^constant]
This equation has an interesting property, the left-hand-side refers to a variable `x`, which does not occur in the right-hand side.

Recall that in any equation, we are always free to bijectively rename all variables: If a property holds for some variables, the property holds for all variables -- no variable is special.
Thus from `x-x = Zero` we can directly infer `y-y = Zero`.
And thus it follows `x-x = y-y`.[^general]

And we want to apply `a = c(x)`. As we are free to rename the variables in our equations, we can also infer `a = c(y)`.
And thus `c(x) = c(y)` from it.


In a slotted e-graph this would mean

```
a := Zero
b(x) := x
c(x) := b(x) - b(x)
```

In other words, one can argue that the e-class `c(x)` contains the e-node `Zero`, whereas `c(y)` also contains the e-node `Zero`, and thus `c(x) = c(y)`.

[^constant]: I write "Zero" to prevent being ambiguous with the numeric variables `0, 1, 2` that we use in shapes.
[^general]: This works generally. If you have an equation `t1 = t2`, where `x` comes up in `t1`, but not in `t2`, you can always derive `t1[x := y] = t2` and thus `t1 = t1[x := y]` (assuming `y` fresh).
