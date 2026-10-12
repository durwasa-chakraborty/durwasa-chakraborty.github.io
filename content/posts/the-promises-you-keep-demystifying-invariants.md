+++
title = 'The Promises You Keep :: Demystifying Invariants'
date = 2026-10-10T00:00:00+05:30
draft = false
author = 'durwasa'
math = true
comments = true
tags = ['programming', 'formal-methods', 'invariants', 'fpl']
+++

Writing invariants gets treated as an art form, which is a polite way of saying nobody can explain how they do it. "Art is long and life is short," said Hippocrates, a man who never had to read a 2,000-word post on formal methods. So I've tried to keep this one short.

![Bare, branching trees dissolving into thick fog, in black and white, with a few silhouetted figures along the bottom edge.](/images/kodaikanal-fog.jpg)

Kodaikanal, 26 January 2024 · Fujifilm X-T3 · XF 33mm f/1.4 · 1/8000 s · ISO 160
{.photo-caption}



The title could have gone anywhere. It could have been a heartfelt review of [*Dorothy*](https://en.wikipedia.org/wiki/Dorothy_(film)), the last Tamizh movie I watched, a beautiful film that is still running in theatres as I write this ( please go and watch it). Instead, it's a formal verification blog. Criticizing cinema, cricket and politics is easy, and everyone with a phone is already doing it, so I will resort to something genuinely difficult: making you care about invariants. 

There are surprisingly mechanical ways to work out what an invariant should be,
though, so let's try to demystify the science behind it.

## The queue

I've been spending time with the Herlihy–Wing queue, the array queue from
Herlihy and Wing's 1990 paper on linearizability. 

```
back  : int           - initially 0
items : array of val  - initially all null

Enq(x):  i := back
         back := back + 1
         items[i] := x

Deq():   for i := 0 to back - 1 do
           if items[i] ≠ null then
             x := items[i]
             items[i] := null
             return x
         return EMPTY
```

`Enq` claims the slot at `back`, moves the marker on, then fills the slot.
`Deq` walks from the front and empties the first slot that holds something.
The marker never moves backwards. 

![Animation: Enq(1) and Enq(2) each claim a slot by advancing back, then fill it; Deq empties slot 0 and returns 1. A row of states builds up underneath.](/images/queue-enq-deq.svg)

## States, and too many of them

A *state* is one snapshot of `(back, items)`, taken between operations, like
each green box above.

Shrink the array to three slots and allow only the values `1` and `2` (yes, a queue that can only ever hold the numbers one and two; just play along, it's a budget production). Each
slot is `NULL`, `1` or `2`, and `back` runs from 0 to 3, so there are
$4 \times 3^3 = 108$ states. That's the mathematical world: every combination,
counted without asking what `Enq` does.

The other world is the runs: the states that some sequence of `Enq` and `Deq`
can actually produce. Each `v` can take either (1 or 2):

| `back` | shapes below the marker                                    | states |
|-------:|------------------------------------------------------------|-------:|
|      0 | `[]`                                                       |      1 |
|      1 | `[NULL]`, `[v]`                                            |  1 + 2 = 3 |
|      2 | `[NULL,NULL]`, `[NULL,v]`, `[v,v]`                         |  1 + 2 + 4 = 7 |
|      3 | `[NULL,NULL,NULL]`, `[NULL,NULL,v]`, `[NULL,v,v]`, `[v,v,v]` |  1 + 2 + 4 + 8 = 15 |
|        |                                                            | **26** |

Each `v` doubles the count. Everything at or past the marker stays `NULL`. 
0
![Two rows of queue snapshots. Reachable: back=2 with [NULL,2,NULL]; back=3 with [NULL,1,2]; back=1 with every slot empty. Unreachable, offending slot ringed: back=1 with [NULL,2,NULL]; back=3 with [1,NULL,2]; back=0 with [1,NULL,NULL].](/images/queue-reachable-states.svg)

Not an official definition, just my _मन की उपज_ (roughly, "something my own
mind cooked up"): an invariant is a funnel between the two worlds. A funnel
with opinions. Pour all 108 states in at the top. Each conjunct you add is
another sieve, and whatever drips out the bottom should be something a queue
could actually be. Herlihy and Wing fitted three sieves. Two of them, it turns
out, have holes you could drive a truck through.

| conjunct added                                                     | states left |
|--------------------------------------------------------------------|------------:|
| (none)                                                             |         108 |
| $\texttt{back} \ge 0$                                              |         108 |
| $\mathrm{low}(\texttt{items}) = 0$                                 |         108 |
| $\forall j \ge \texttt{back}.\; \texttt{items}[j] = \texttt{null}$ |          40 |
| *reached by some run*                                              |          26 |

The first two cut nothing here, because we assumed both when we counted:
`back` started at 0 and the array at index 0. The third conjunct does the cutting. It removes all 68 states
with a value at or past the marker, the kind ringed in the red row above.

We could count here because the queue only has three slots. Give it an
unbounded array and counting stops working: the set of good states may be
infinite. We still want to say that every run stays inside the good set (the
_40_ in our toy), and we don't want to check that one state at a time. So
instead we check two things:

1. The empty queue is in the 40.
2. If you're in the 40 and do one `Enq` or `Deq`, you're still in the 40.

Together these cover every run. The first state is in, and each step keeps
the next one in. That's induction. 

But wait. The same two checks pass for the 108:

1. The empty queue is in the 108.
2. If you're in the 108 and do one `Enq` or `Deq`, you're still in the 108.

They pass trivially, because the 108 is every state the toy can be in. So what
stops us from using `true`?
The trouble is what it lets in. The 108
includes `back = 1, [NULL, 2, NULL]`, the impossible state from the red row. Run
`Enq(1)` from there. It writes at `back`, steps on the `2`, and leaves
`back = 2, [NULL, 1, NULL]`.

That's still one of the 108, so check 2 passes. What broke is the specification of a queue. It
held `2`, we enqueued `1`, and now it holds only `1`. `Enq` didn't append; it
overwrote. An invariant that lets `back = 1, [NULL, 2, NULL]` in can't rule that out,
so it can't help you prove that `Enq` appends.

The 40 keeps it out. In every one of the 40, the slot at `back` is empty, so
`Enq` never has anything to step on. Both invariants survive the two checks,
but only one is strong enough to prove the queue is a queue.

## A queue, and a goal that won't close

**Definition (abstraction function).** For a representation
$r = (\texttt{back}, \texttt{items})$, the abstract queue it denotes is the
set of its occupied slots, read in index order:

$$A(r) \;=\; \{\, (j, \texttt{items}[j]) \;\mid\; \texttt{items}[j] \mathrel{\texttt{!=}} \texttt{NULL} \,\}.$$

**Definition (representation invariant).** A predicate $I$ on representations
such that $I(r_0)$ holds for the empty queue $r_0$, and $I(r) \Rightarrow
I(\mathit{op}(r))$ for every operation $\mathit{op}$.

**Claim (`Enq` appends).** Let $r' = \mathrm{Enq}_x(r)$ and $i = r.\texttt{back}$. Then

$$I(r) \;\Longrightarrow\; A(r') \;=\; A(r) \cup \{(i, x)\}.$$

The invariant is the only bridge between the two sides. $A(r)$ and $A(r')$
are computed from two different arrays, and whatever we know about how they
relate has to come from $I(r)$:

```
        r  ───── Enq(x) ─────▶  r'
        │                        │
        A                        A
        ▼                        ▼
      A(r)  ── ∪ {(i, x)} ──▶  A(r')        holds only given I(r)
```

**Attempt** with $I(r) \;\equiv\; \texttt{back} \ge 0$.

This is a valid invariant: $\texttt{back}$ starts at $0$ and is only ever
incremented. Unfold `Enq`. It sets $\texttt{items}' = \texttt{items}[i \mapsto x]$
and $\texttt{back}' = i + 1$, so slot $i$ is overwritten and

$$A(r') \;=\; \bigl(A(r) \setminus \{(i, \texttt{items}[i])\}\bigr) \;\cup\; \{(i, x)\}.$$


This is the remaining goal, and it can't be reduced any further. The only
hypothesis is $\texttt{back} \ge 0$, which says nothing about
$\texttt{items}$. No rewrite applies, and $\texttt{items}[i]$ stays as it is.

Nor would a cleverer argument help, because under this hypothesis the claim is
false. Take $r = (1, [\texttt{NULL}, 2, \texttt{NULL}])$. It satisfies
$\texttt{back} \ge 0$, but $\mathrm{Enq}_1$ overwrites the $2$:
$A(r') = \{(1, 1)\}$, while $A(r) \cup \{(1, 1)\} = \{(1, 2), (1, 1)\}$.

The goal is not just underivable from $\texttt{back} \ge 0$; it is semantically invalid, because a countermodel exists.

## The invariant that discharges it

Strengthen the invariant with the missing fact:

$$I(r) \;\equiv\; \texttt{back} \ge 0 \;\wedge\; \forall j \ge \texttt{back}.\; \texttt{items}[j] = \texttt{NULL}.$$

**Proof (`Enq` appends).** Let $r$ satisfy $I$, write $i = \texttt{back}$,
and let $r' = \mathrm{Enq}_x(r)$. One step at a time:

$$
\begin{aligned}
A(r') &= \bigl(A(r) \setminus \{(i, \texttt{items}[i])\}\bigr) \cup \{(i, x)\}
  && \text{Enq overwrites slot } i \text{ with } x \\[4pt]
&= \bigl(A(r) \setminus \{(i, \texttt{NULL})\}\bigr) \cup \{(i, x)\}
  && I(r) \text{ at } j = i, \text{ since } i \ge \texttt{back} \\[4pt]
&= A(r) \cup \{(i, x)\}
  && (i, \texttt{NULL}) \text{ is not in } A(r)
\end{aligned}
$$

The last step holds because $A(r)$ only collects occupied slots, so taking
away a `NULL` pair takes away nothing.

That shows `Enq` *adds* $x$. It remains to show $x$ lands at the *back*,
behind everything already queued. Read the invariant the other way round:
an occupied slot can't be at or past the marker, so

$$\texttt{items}[j] \mathrel{\texttt{!=}} \texttt{NULL} \;\Longrightarrow\; j < \texttt{back} = i.$$

Every index in $A(r)$ is below $i$, so $(i, x)$ comes last in index order.
And since $\texttt{back}$ only ever grows, the next `Enq` gets index $i + 1$,
behind $x$. The order is kept for good. $\blacksquare$

In Lean 4, the new conjunct shows up in `ih`, and instantiating it at
`r.back` closes the goal that was stuck before:

```lean
case enq
x  : Val
r  : Rep
ih : r.back ≥ 0 ∧ ∀ j ≥ r.back, r.items j = none
⊢ r.items r.back = none
```

You don't guess the invariant. You read it off the failure: the stuck goal
$\texttt{items}[\texttt{back}] = \texttt{NULL}$ named exactly the fact that was
missing.

## Exercise: selection sort

Your turn. Here's selection sort on an array $a$ of length $n$:

```
for i := 0 to n - 2 do
  m := i
  for j := i + 1 to n - 1 do
    if a[j] < a[m] then m := j
  swap a[i], a[m]
```

Each pass finds the smallest element of $a[i..n)$ and swaps it into slot $i$.
Fill in the blank so that $I(i)$ holds at the top of every pass and is strong
enough to prove the array ends up sorted:

$$I(i) \;\equiv\; \mathrm{sorted}(a[0..i)) \;\wedge\; \boxed{\quad ? \quad} \;\wedge\; \mathrm{perm}(a, a_0)$$

Here $a[0..i)$ is the prefix $a[0], \dots, a[i-1]$, and
$\mathrm{perm}(a, a_0)$ says $a$ is still a rearrangement of the original
array $a_0$. Pick one and answer in the comment section:

- **(A)** $\forall p < i.\; a[p] \le a[i]$
- **(B)** $\forall p < i \le q < n.\; a[p] \le a[q]$
- **(C)** $\mathrm{sorted}(a[i..n))$
- **(D)** $a[i] = \min a[0..n)$

Do it the way this post did: start with the weakest invariant you can, try the
proof, and let the stuck goal tell you what's missing. The hints walk through
that, one step at a time. Open them only as far as you need.

<details>
<summary><strong>Hint 1</strong>: what has to be proved</summary>

Two obligations. Preservation, for one pass of the outer loop:

$$I(i) \;\wedge\; \text{one pass} \;\Longrightarrow\; I(i+1)$$

and exit: the loop stops with $i = n - 1$, so

$$I(n-1) \;\Longrightarrow\; \mathrm{sorted}(a[0..n)).$$

</details>

<details>
<summary><strong>Hint 2</strong>: try the weak invariant first</summary>

Drop the blank and take $I(i) \equiv \mathrm{sorted}(a[0..i))$ on its own.
After the inner loop, $a[m]$ is the minimum of the suffix, and the swap moves
it into slot $i$:

$$a'[i] \;=\; \min_{\,i \le q < n} a[q].$$

The new prefix is the old one with $a'[i]$ appended. So
$\mathrm{sorted}(a'[0..i+1))$ comes down to one comparison:

$$\mathrm{sorted}(a'[0..i+1)) \;\Longleftrightarrow\; \mathrm{sorted}(a[0..i)) \;\wedge\; a[i-1] \le a'[i].$$

(For $i > 0$. At $i = 0$ there's nothing before slot $0$ to compare with.)

</details>

<details>
<summary><strong>Hint 3</strong>: the stuck goal</summary>

Substitute the value of $a'[i]$, and the goal that's left is

$$a[i-1] \;\le\; \min_{\,i \le q < n} a[q].$$

In Lean 4:

```lean
case pass
a  : Array Int
i  : Nat
ih : Sorted (a.extract 0 i)
⊢ a[i-1] ≤ a[i:].min
```

`ih` is a fact about the prefix only. The goal compares the end of the prefix
with the suffix. Which option gives you a fact that crosses the boundary at
$i$?

</details>

<details>
<summary><strong>Hint 4</strong>: check your pick against both obligations</summary>

A correct blank has to survive three tests:

1. It discharges the stuck goal from Hint 3.
2. It is itself preserved by a pass: assuming it at $i$, prove it at $i + 1$.
   Remember the swap only moves elements *within* the suffix
   $a[i..n)$.
3. It makes the exit work: at $i = n - 1$, the prefix $a[0..n-1)$ is sorted,
   but you still need

   $$a[n-2] \;\le\; a[n-1].$$

One of the four options gets through all three. At least one of the others is
not even true of a real run. Try it on $a_0 = [3, 1, 2]$.

</details>
