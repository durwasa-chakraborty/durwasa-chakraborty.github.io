+++
title = 'Data Races'
date = 2025-04-06T18:50:32+05:30
draft = false
author = 'durwasa'
math = true
comments = true
tags = ['programming', 'systems']
+++


*“Data races are bad.”* — Every systems programming course, ever.  
But **why** are they bad? And **what** exactly are they?

Let’s be a little Aristotelian about this—**question everything**. So here we go:

---

### So... What *Is* a Data Race?


> A **data race** occurs when two or more concurrent threads access the **same memory location** , at least one of them performs a **write**, and there's **no synchronization mechanism** (like a lock or atomic operation) protecting that access.

Whew. That’s a lot of words. Let’s unpack that.


* “Two or more threads” → We're dealing with a multithreaded (or concurrent) program. A single-threaded program is too well-behaved to ever encounter a data race.

* “Same memory location” → Both threads are peeking (or poking) at the same variable.

* “At least one write” → If both threads are just reading, all’s peaceful. But the moment one tries to write—or update—the memory, conflict brews.

* “No synchronization” → No locks, no atomics, no coordination. It’s the memory access version of “every thread for itself.”

Put all of that together, and you’ve got yourself a classic case of undefined behavior, kicking down the door and yelling:
“You summoned me? Initiating FireRocket().”

---

### Dekker’s Algorithm 


Just two threads doing two things each.

### Initial Setup

We have two shared variables:
```
int x = 0;
int y = 0;

```

And two threads:

#### Thread A:
```c
x = 1;
r1 = y;
```

#### Thread B:
```c
y = 1;
r2 = x;
```

That's it. No mutex. No atomic. Just pure, unfiltered **concurrent drama**.



### What Should Be the Output?

Let’s think like a sequential human being.

If Thread A runs fully before Thread B:
```text
x = 1;
r1 = y;   // r1 = 0
----------------
y = 1;
r2 = x;   // r2 = 1
=> r1 = 0, r2 = 1
```

If Thread B runs fully before Thread A:
```text
y = 1;
r2 = x;   // r2 = 0
----------------
x = 1;
r1 = y;   // r1 = 1
=> r1 = 1, r2 = 0
```

If both writes happen before both reads:
```text
x = 1;
y = 1;
r1 = y;   // r1 = 1
r2 = x;   // r2 = 1
=> r1 = 1, r2 = 1
```

So our **intuitive, reasonable human outputs** are:

- `(r1, r2) = (0, 1)`
- `(r1, r2) = (1, 0)`
- `(r1, r2) = (1, 1)`

All of these are fine and expected.

### But Wait—A Quick Math Detour 

Let’s get mathematical for a second. If you have two threads and each one performs two operations (a write and a read), the total number of **interleavings** of those operations is given by:

$$
\binom{2 + 2}{2} = \frac{(2 + 2)!}{2! \cdot 2!} = \frac{4!}{2! \cdot 2!} = \frac{24}{4} = 6
$$

So there are **6 valid interleavings** of the operations from Thread A and Thread B.

These correspond to permutations where the **relative order of operations within each thread is preserved**. That is:

- `x = 1` before `r1 = y` (Thread A’s order preserved)
- `y = 1` before `r2 = x` (Thread B’s order preserved)

Let’s list a few examples:

1. A1 A2 B1 B2 → `(r1 = 0, r2 = 1)`
2. A1 B1 A2 B2 → `(r1 = 1, r2 = 1)`
3. B1 B2 A1 A2 → `(r1 = 1, r2 = 0)`
4. B1 A1 B2 A2 → and so on…

These 6 interleavings give rise to our **expected outputs**:  
`(0,1)`, `(1,0)`, and `(1,1)`.

In general ::
For $n$ threads, where each thread performs $o_1, o_2, \cdot o_n$ memory operations,
$$
\text{Total Interleavings} = \frac{(o_1 + o_2 + \ldots + o_n)!}{o_1! \cdot o_2! \cdots o_n!}
$$





---

### So What’s the Problem?

That nasty little `(r1, r2) = (0, 0)` doesn’t appear in any of the above 6 valid interleavings!

Why? Because it would require **reading both `x` and `y` before either write has occurred**, which **violates the intra-thread ordering** unless reads were moved *before* writes.

This is why:

> **Reordering breaks the mathematical expectation** based on interleavings and causes outputs that should be **impossible under sequential consistency**.

So while our math predicted 6 nice, clean outcomes, the CPU/compiler pulls a little *abracadabra*, breaks the sequential spell, and suddenly:

```text
(r1, r2) = (0, 0)

```

### But Here’s the Twist... 

Suppose your compiler or hardware chooses to reorder instructions, perhaps assuming it knows best. (For example, reordering a load $\rightarrow$  a store, among other classic computer architecture behaviors—which I’ll delve into in a separate post.)

Imagine this:

1. Both threads reorder the **read before write**.
2. Now, Thread A reads `y` (which is still 0), then writes `x = 1`.
3. Simultaneously, Thread B reads `x` (also still 0), then writes `y = 1`.

So we get:
```text
r1 = y;   // 0
r2 = x;   // 0
x = 1;
y = 1;
=> r1 = 0, r2 = 0   ← the "impossible" outcome!
```



But that shouldn’t be possible, right?

If either thread had seen the other's write, one of the reads should be 1.  
Yet due to **reordering** and **no synchronization**, both reads saw 0.

--- 

This output:

```text
(r1, r2) = (0, 0)
```

**should** be impossible under **sequential consistency**.  
But under **real-world hardware + compiler optimizations**, it *can* happen.


### What is sequential consistency now? 

Leslie Lamport defines **sequential consistency** as:

> *"The result of any execution is the same as if the operations of all the processors were executed in some sequential order, and the operations of each individual processor appear in this sequence in the order issued."*

**Translation**: It should feel like *all memory operations from all threads were executed in some sequential order*, even if they were running concurrently.

$$
\forall a, b \in S,\ a \leq b\ \vee\ b \leq a
$$


For any two processes 
$p$ and $q$ you can always determine which one comes before the other (or that they are the same).

This contrasts with partial orders, where some processes might be incomparable like concurrent processes in a distributed system with no causal link.
This implies a **total order** of all operations. That is, everything can be lined up in a single straight line.  


If you only had **partial order** (like Thread A and Thread B doing stuff independently), you'd never know which came first.  
Which is great for eggs, terrible for memory consistency.


### Dear Compiler: Please Stop Being So Smart


I mean, seriously. I wrote my code like this:

```c
x = 1;
y = 2;
```

I want you to do **exactly that**. Line by line.  
No backflips, no clever tricks, no playing 4D chess with my instructions.

I want you to translate it into assembly like a faithful scribe from an ecclesiastical journal—quietly, obediently, reverently.  
Just copy it down, convert into assembly, binary, the works!

I’ve got the world’s biggest SSD, a GPU that can simulate *Interstellar* in 16K, and enough RAM to mine all the bitcoins—twice.  
**Optimization is not my concern. I want correctness.**

So what’s the problem?

Well, here’s the catch: **correctness isn’t the only thing you care about**—even if you think it is.  
Because guess what? Turning off every optimization doesn’t magically fix **data races** or **memory consistency issues**.  
That bug you blamed on “compiler wizardry”?  
**Yeah, that’s on you, buddy.**

---

Modern processors are sneaky little devils. They reorder instructions under the hood.  
They prefetch, speculate, parallelize—all while laughing in binary.  
Even if the compiler leaves your code untouched, the hardware might still go, “eh, close enough.”

So unless you explicitly tell both the compiler **and** the CPU:

> “Hey, don’t mess this up—I mean it,”

you're still rolling the dice.

And if you think `-O0` is your silver bullet?

Well... **welcome to the illusion of safety.**

---

### Let’s Make That Concrete 

Here's a simple-looking C++ example:

```cpp
int flag = 0;
int data = 0;

void writer() {
    data = 42;
    flag = 1;
}

void reader() {
    if (flag == 1) {
        printf("%d\n", data);
    }
}
```

Two threads. `writer()` sets data, then signals with `flag`. `reader()` checks `flag` and reads `data`.

Looks innocent, right? But this code has a **data race**.  
No locks, no atomics, no memory fences.

Even with `-O0`, the compiler is **allowed to reorder** instructions because C++ says: *“you gave me undefined behavior, so all bets are off.”*

---

### So What Happens?

Maybe `flag = 1` gets reordered before `data = 42`.

Maybe `reader()` sees `flag == 1`, but reads **stale or garbage** from `data`.

It might run flawlessly on your machine, but crashes on the production server at 3AM—triggering a page and dragging you out of bed at an ungodly hour.

---

### Fix It the Right Way 

```cpp
#include <atomic>

std::atomic<int> flag{0};
int data = 0;

void writer() {
    data = 42;
    flag.store(1, std::memory_order_release);
}

void reader() {
    if (flag.load(std::memory_order_acquire) == 1) {
        printf("%d\n", data);
    }
}
```

This version uses proper synchronization:  
- `memory_order_release` in `writer()` ensures `data` is visible before `flag` is updated.
- `memory_order_acquire` in `reader()` ensures `data` is read **after** confirming `flag`.

---

### Bottom Line?

**Correctness isn’t passive.**  
You don’t get it by turning knobs like `-O0`. You earn it by using the right tools—locks, atomics, memory barriers.

Because neither your compiler nor your CPU is in the business of babysitting your assumptions.

Welcome to the real world!

You wanted a **cup of water**.  
The compiler optimization will  give you a *caramel swirl, three skim, one sugar* — and maybe some regret.  {{<mention "Courtesy Dunkin Donuts" "https://www.instagram.com/p/CZ0RjLzFOMX/">}}

In the next part we will cover - Memory model of Programming Languages.

---
