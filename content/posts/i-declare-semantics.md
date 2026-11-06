+++
title = 'I Declare Semantics :: What Is Declarative Semantics'
date = 2026-10-30T00:00:00+05:30
draft = true
author = 'durwasa'
math = true
comments = true
tags = ['programming', 'formal-methods', 'semantics', 'fpl']
+++

![Two people seen from behind on a metal bench at the rim of the Grand Canyon, the layered canyon walls fading into haze, in black and white.](/images/couple-at-grand-canyon.jpg "Carved By Time; Shared by two")

Grand Canyon, Arizona, USA, 10 January 2018 · Fujifilm X-T3 · XF 56mm f/1.2 · f/5 · 1/1100 s · ISO 160
{.photo-caption}

I DECLARE ~~BANKRUPTCY~~ SEMANTICS! 

{{< mention "Reference : Michael Scott Declaring Bankruptcy" "https://www.youtube.com/watch?v=C-m3RtoguAQ" >}}

After my _phenomenal success_ of my last blog that almost competed with Rajni sir's [Jailer 2](https://en.wikipedia.org/wiki/Jailer_2) at the box office, if the box office is my WhatsApp/Discord and the collection is counted in people reaching out of 'nowhere' to text me that they are reading the blog. I am extremely; extremely happy!

But. Most of my friend circle are in business, entrepreneurs, musicians and even within the computer science landscape formal verification is a relatively new idea. So a lot of the heartfelt messages went like: "I read it, but I didn't understand the technical part." And as {{< mention "Benjen Stark" "https://gameofthrones.fandom.com/wiki/Benjen_Stark" >}} once said, nothing someone says before the word "but" really counts. So the actual review was *didn't understand the technical part*. Ouch! Let's be extremely critical of my own explanation and think about how to make the material more accessible. :) 

So today we will look into declarative semantics.

## Operational Semantics: Watching The Gears Move

What is declarative semantics, and what does it mean as opposed to operational semantics?

Operational semantics is about how things *operationally* move.  Prof. {{< mention "V. Krishna Nandivada" "https://www.cse.iitm.ac.in/~krishna/" >}}  once told me that you cannot explain points-to analysis using "points to", or any verb form of it. I have just committed the same felony. So let me try again without the letters o-p-e-r-a-t-i-o-n: operational semantics describes a program by the steps a machine takes to run it, one step at a time, from the input all the way to the answer.

We will keep it very simple, throughout the blog. Say we have an `add` function. It takes two 'things' and returns their sum.

To give `add` an operational semantics, you need to know how every individual gear of this adding machine moves. You might build it out of gates: NAND gates alone, or some minimal configuration of AND, OR and NOT gates, the kind you get out of a K-map. (courtesy {{< mention "Arnav Pahuja" "https://www.linkedin.com/in/arnav-pahuja/" >}})  or you can just have `add(1, 2)` return `x + y`, where the `+` is secretly an `addq` or some other assembly-level detail. Whichever you pick, at each step you are telling the machine exactly how to take the next step.

Now, given my glittering record in Computer Architecture (I topped the class, if you sort the marks in ascending order), I will stay as far away from gates as I can. My other option is lambda calculus, but it is too close to my heart to be squeezed into a corner of a post about declarative semantics. It deserves a post of its own. Besides in the history of mankind no paragraph containing a λ and a latin word for a pebble (_calculus_) has ever been described as non-technical. And if I start on lambda calculus now, I will lose the three readers I gained from the last post.

## The Oracle And The Holy Book

What do all oracles and Gods have in common? The Gods give you a book or something similar to follow, and the book has axioms. Sometimes these axiomatic specifications are wrong, sometimes they contradict each other, and those gaps are exactly where certain individuals get to do nefarious things in the name of the prescribed book!

Now suppose you ask the aforementioned Oracle "what is `add(2, 3)`?" and it gives you an answer. No gears, no steps, just an answer. How do you know it is right?

Whatever answer the oracle gives us, we run it past the axioms. If it obeys every one of them, it is accepted. We never once ask *how* the oracle got there.

## Let's Start A Religion (Since We Don't Have Enough Of Them)

Alright, so let's make our own small religion whose only God is addition (or shall I say, let's {{< mention "proselytize" "https://www.merriam-webster.com/dictionary/proselytize" "Merriam-Webster: to induce someone to convert to one's faith" >}} :P).

The congregation of suspects: `add`, `sub`, `div`, `mult`, `max` and `min`. Our job is to write commandments until only `add` is left standing.

**Commandment 1.** `add(a, b) = add(b, a)`.

`sub` is excommunicated on day one: `sub(5, 3) = 2`, but `sub(3, 5) = -2`. Swap the arguments and the answer changes its sign, which in any religion is a sign of a shifty character. `mult`, `max` and `min`, on the other hand, are perfectly happy to swap their arguments, so they get to stay in the congregation. Division is shown the door right behind `sub`: `div(6, 3) = 2` but `div(3, 6) = 0.5` Either way, *division* is not the point of the religion. :D


**Commandment 2**, `add(a, 0) = a`.

Now `mult` is out (`mult(5, 0) = 0`) and so is `min` (`min(5, 0) = 0`). But if our domain is the natural numbers $\mathbb{N} = \\{0, 1, 2, \ldots\\}$, then `max(a, 0) = a` as well. We still risk `add` and `max` being the same god. We don't want that, right?

So we wait for a wise hermit, who brings a successor function, $S(n) = n + 1$, and a new commandment: `add(a, S(b)) = S(add(a, b))`.

Wait!! If you don't have `+` `add`, then how did you define $S(n)$ as $n + 1$? Our religion depends on an oracle; the oracle can't depend on another religion! We need some base God on which the system as a whole can stand.

$S$ is not defined in terms of anything. $S$ just *is*. The numbers are $0$, $S(0)$, $S(S(0))$, and so on; 3 is not a thing in itself, 3 is just a representation of $S(S(S(0)))$. Nobody added anything to make $S$.

With $S$ in hand, `max` is finally out: `max(5, S(0)) = 5`, but `S(max(5, 0)) = S(5) = 6`. Excommunicated.

### The Scripture

So now the `add` function has exactly two axioms:

```text
add(a, 0)    = a
add(a, S(b)) = S(add(a, b))
```

And that is it. These two lines declaratively define `add`. Any function on $\mathbb{N}$ that obeys them *is* addition; there is no room left for an impostor. Here is $2 + 1$, checked rather than computed:

```text
add(S(S(0)), S(0))
  = S(add(S(S(0)), 0))      -- successor commandment
  = S(S(S(0)))              -- commandment 2
  = 3
```

If that went by too fast, here it is in slow motion. Notice that the engine never adds anything. It only holds the term up against each commandment, checks whether the shapes line up, and moves pieces around. Press *Next step*.

{{< match-steps >}}

The plot twist: with these two lines, Commandment 1 is no longer needed. You can *prove* `add(a, b) = add(b, a)` from them by induction. Our founding commandment got demoted to a theorem. Organised religion, I tell you.

You don't need to know how `add` is implemented. NAND gates, what gates, not gates, QuantumFT adder, lambda calculus, some API call to `https://api.totally-legit-adder.ai/v3/add?a=2&b=3&api_key=sk_live_pls_dont_leak_4f9a2c` that just adds two numbers, or a JavaScript function (where `'2' + 3` is `'23'`, which our axioms will reject very swiftly). You don't even need to know how `a` and `b` are stored: bits, strings, beads on an abacus. As long as they behave like $0$ and $S$, the axioms take care of everything.

At this point every Computer Science student thinks: "But what if `a` is `INT_MAX` and `b` is 1? Integer overflow!" Integer overflow is a very worldly problem, and oracles do not concern themselves with worldly problems. The more scientific answer is that our religion has no `INT_MAX`. A number is just a string of `S`s, brackets and a `0`: 3 is `S(S(S(0)))`, and a bigger number is just a longer string. Strings can be written on a tape, and the tape (Turing's, not Sellotape(which I recently came to know is not 'Cellotape')) never runs out. And if your oracle *does* secretly run on 32-bit integers, the axioms will catch it. They demand that `add(INT_MAX, S(0))` be exactly one more than `INT_MAX`, and the machine will answer with a large negative number if it is in a good mood (in C, signed overflow is undefined behaviour, so it does not have to be in a good mood). Caught lying. Excommunicated.

That is declarative semantics. It lets you reason about a program as a *definition*, without worrying about how the gears move, or even what the gears are.

## Back To Real Programs

Clearly, in programming we are doing much more than just writing an `add` function. But we do define properties declaratively, as specifications:

- Once I order an Uber and a driver has locked in, I don't want any other Uber driver to pick up that user. At most one driver per ride. Ever.
- Once I have invested 100 USD in the next generation of HFT and the money has been deducted, I want it to show up in my total balance. Not "eventually". Not "probably".
- Once my priority queue has popped its top element $e$, I want $e \notin pq$. (Unless it was in there twice. See, writing specifications is hard.)
- Once the list is sorted, I want the result to be in order *and* to be a permutation of the original list. Just "in order" is not enough: return `[]` and you have a beautifully sorted list.

None of these say *how*. They only say *what must be true*. Each one is a commandment, the implementation is the oracle, and our job is to check that the oracle obeys. If that sounds familiar, it is because it is exactly what I was doing with [invariants]({{< ref "the-promises-you-keep-demystifying-invariants" >}}) in the last post.

## Amendments

Well, there is one difference. Nobody amends a holy book, and well, I would not want them to.  Scripture is meant to be eternal, and that is rather the point of it. Whenever mortal beings make something amendable, the amending passes through a power imbalance: whoever has the numbers, the wealth or the position starts rewriting it to suit themselves. A book that nobody can amend cannot be bent by anyone, whatever the social order of the day happens to be. Its finality is what keeps it out of everyone's hands, equally.

Science works the other way round. You can be a lowly clerk at a patent office in Bern, write a paper telling a certain Lucasian Professor of Mathematics at Cambridge, the greatest scientific mind of all time, that his *Philosophiae Naturalis Principia Mathematica* is not the final word, and get your amendment accepted. :) And to quote Karl Popper: a theory that nothing could ever refute is not science at all. Being refutable is not a weakness of science; it is the definition of it.

Specifications live on the science side of that line. When a spec turns out to be wrong, the way "the list is in order" happily let an empty list through, we do not excommunicate the person who noticed. We amend the spec.



---

### References & Links

- **Peano axioms** :: [Wikipedia](https://en.wikipedia.org/wiki/Peano_axioms) :: the $0$ and $S$ our little religion of addition is built on
- **Bill Bryson** :: [*A Short History of Nearly Everything*](https://en.wikipedia.org/wiki/A_Short_History_of_Nearly_Everything), 2003, p. 74 :: the mentioning of Isaac Newton as Lucasian Professor of Mathematics at University of Cambridge.
