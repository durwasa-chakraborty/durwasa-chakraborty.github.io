+++
title = 'Prof. CPR and the Resuscitation of Algorithmic Thinking'
date = 2026-05-09T01:06:10+05:30
draft = false
author = 'durwasa'
math = true
comments = true
tags = ['programming', 'algorithmic-thinking', 'pedagogy', 'essay', 'fpl']
+++



![CPR's board, IIT Madras](/images/cpr_lecture_wall.JPG)

> *"You start problem-solving with mathematical thinking. To obtain a solution, you extend it to computational thinking. 
> But to actually implement it on the computer, you  need
> algorithmic thinking"* 
> Prof. CPR

> **A note about this article.** Most of the body is a faithful transcript of Prof.
> CPR's lecture as I audio recorded it; the worked examples, asides, and reasoning
> detours are my own interpretation. Every SVG figure and code snippet was
> generated with claude-opus-4.7.


 A wave of energy walked into the room with {{< mention "Prof. Shwetha Agarwal"
   "https://www.cse.iitm.ac.in/~shwetaag/" >}} - this year's Vigyan Puruskar
   recipient, Erdős number of 3, and supremely humble to carry it more casually than someone like me who has an array of unverified academic delusions.
   She rose to introduce our guest lecturer, the man who had come to inaugurate {{< mention "ARTCS"
   "https://artcs.iitm.ac.in/" >}}, the Arvind Raghunathan Institute for
   Theoretical Computer Science.

   That was {{< mention "Prof. C. Pandu Rangan"
  "https://en.wikipedia.org/wiki/C._Pandu_Rangan" >}} - affectionately known to
   as Prof. CPR. The almighty, has a serious sense of humour about initials. Over the next ninety minutes he resuscitated a way of thinking that an entire
  generation has begun quietly outsourcing to an AI chatbot. And a 
  generation before it (technically my generation), too impatient to dig down into seminal works, was happy
  to swap for the first plausible answer on Stack Overflow.
  
   
  ---
    
  History is one of the great pedagogies for a reason. It teaches
  us discipline of resource scarcity, amongst other things. It took roughly 4
  kilobytes of RAM to put two human beings on the Moon. I, in 2026 with 16GB RAM, will
  complain that my REST API feels sluggish and blame it on my laptop's RAM, when
  the API is doing nothing more demanding than rendering a JSON list.
                                                            
  I still remember the Class XI NCERT chemistry textbook, where Dalton’s atomic model appeared with all the scientific dignity of a watermelon approximation. Naturally, I asked my chemistry teacher, {{< mention "Vibhawari Choubey" "https://www.facebook.com/vibhawarichoubey" >}}
  why on earth we had to study something so gloriously incorrect when modern science had clearly moved on from produce-based examples to drive a point.

She replied, in the kind of quiet voice teachers reserve for moments of philosophical assassination:
_“You cannot laugh at the bizarreness of Dalton’s model while standing on Dalton’s shoulders.”_


NCERT, for all its quirks, seemed to understand something I deeply miss in many
modern scientific textbooks: science is not merely a polished archive of final
answers, but a living chronology of human curiosity.

To be fair, especially in much of Western technical authorship, textbooks are
often designed exactly as they should be ; precise, efficient, sharply
engineered for clarity. They are optimized to teach the law, the theorem, the
mechanism, the result. And for an engineer, that directness is invaluable.

But I miss the stories.

I miss knowing how an idea was born in uncertainty, how brilliant people fumbled
toward truth through models that were sometimes absurd, sometimes elegant, and
often gloriously incomplete. I miss the origin stories: not just what we know,
but how we came to know it.

The narrative matters. The meandering, overcomplicated, occasionally
sleep-inducing academic bedtime story matters.

It transforms formulas from commandments into discoveries. It gives concepts
ancestry. It lets you see science not as divine revelation handed down in bullet
points, but as humanity’s long sequence of increasingly refined guesses.

And perhaps that is why those stories stay with us longer: because we do not
merely memorize conclusions.

Alright, enough sociology ; lets look into mathematics.

---

## Three modes of thinking

Consider a problem whose solution space has size $\binom{n}{0} + \binom{n}{1} +
\binom{n}{2} + \cdots + \binom{n}{n} = 2^n$. The mathematician inspects this and
pronounces the matter settled: *a solution exists*.

This is the plane on which the existence is the deliverable **mathematical
thinking**. A proof concludes the instant a solution is shown to *be*,
regardless of whether enumerating it would outlast the sun.

Algorithms, however, must eventually run on machines, and machines have clocks,
caches, cores, and a finite tolerance for $2^n$. The moment those constraints
enter the room, you have crossed into **computational thinking**

The bridge between the two is **algorithmic thinking**: the craft of taking a
mathematician's existence proof into a form a machine can finish.



## The articulation-point example: Prim's Algorithm

Cut vertex (articulation point), the warm-up case study.

![Cut vertex](/images/cut-vertex.svg)

### The naive characterization is exponentially bad

A vertex $b$ is a cut vertex iff there exist $x, y$ such that *every*
path from $x$ to $y$ goes through $b$. Beautiful proof ;
computationally hopeless: the number of paths can be exponential, and
you'd have to check $\binom{n}{2}$ pairs or $({n}C_{2})$.

(For the uninitiated: you'd start at $x$, try to walk to $y$, and check whether $b$ shows up on the way on each of them; miss a single $b$ free path and the 
property falls.)


## The MST example: Prim, and the change of viewpoint

The lecture's centrepiece and the part of the board that's most
worth re-reading.

![Cut Parition](/images/cut-partition.svg)

We are given an undirected, connected, weighted graph $G = (V, E, w)$,
arbitrary edge weights, and we want a minimum-weight spanning tree.

### Setup: cuts and the cut property

![Cut Respects A](/images/cut-respects-A.svg)
A *cut* is a partition $(S, V \setminus S)$ of the vertices. An edge
is either *internal* to one side, or *crossing* the partition.

The cut property: the mathematical lever the whole algorithm rests on:

$$
\begin{aligned}
&\text{Let } A \subseteq E \text{ be a subset of some MST.} \\
&\text{Let } C = (S, V \setminus S) \text{ be a cut that respects } A \\
&\text{(no edge of } A \text{ crosses } C\text{).} \\
&\text{Then any minimum-weight crossing edge of } C \\
&\text{can be added to } A,\ \text{and } A \text{ remains extendible to an MST.}
\end{aligned}
$$

Two letters in that statement carry the algorithm's entire weight: $A$ and the cut $C$. 

**Why define $A$ at all?** Because the entire point of greedy MST construction is that it is *incremental*. We do not pull an MST out of the air; we build it one edge at a time. $A$ is the algorithm's running state "the MST so far." It starts empty. It ends with $n - 1$ edges spanning every vertex. Every iteration consists of the same act: pick one more edge to add to $A$ maintainging the following invariant - 

$$
A \text{ is extendible to an MST} \quad (\exists\ \text{MST}\ T^* \text{ with } A \subseteq T^*).
$$



### Step 1: what goes wrong if A contains a crossing edge

Before defending the "respects $A$" clause in words, let's break it in a picture and see what happens.

![A crossing edge in A creates a cycle](/images/cut-violates-A.svg)

Pretend we did not notice the violation and follow the recipe: find the lightest crossing edge of the cut and add it to $A$. Say that edge is $5\text{–}6$ (the solid orange line at the bottom).

Now trace the four edges $1\text{–}6$, $1\text{–}4$, $4\text{–}5$, $5\text{–}6$ on the right panel. They form the shaded loop $1 \to 4 \to 5 \to 6 \to 1$ a four-vertex cycle. Two of its sides cross the cut ($1\text{–}4$ and $5\text{–}6$), two are internal ($1\text{–}6$ and $4\text{–}5$), and together they walk in a circle.

**What this costs us.** A spanning tree has no cycles, so the new $A$ cannot be extended into a spanning tree at all  let alone into a minimum one. 



### Step 2: watching A grow into a spanning tree

![Prim's algorithm animated step-by-step](/images/prim-step2-animation.svg)

Each loop of the animation runs Prim on the same six-vertex graph: vertices flip from red ($V \setminus S$) to green ($S$) one at a time, and the orange-haloed edge is whichever crossing edge just joined $A$ (blue).

The cut $(S, V \setminus S)$ ,visible as the right-hand partition, never repeats, and it always respects $A$ automatically, because $A$ lives entirely inside $S$.

### Step 3: why one-step greedy never costs us globally

**TL;DR.** The locally lightest crossing edge is always inside *some* MST. Any MST that omits it can be rewritten to include it :: swap the light edge in, drop the cycle's other crossing edge out and the result is no heavier. So Prim's myopic pick is globally safe; no "I'll pay $3$ now to save $8$ later" trap can ever punish you.

**Longer Version**

Step 2 watched Prim succeed on a particular six-vertex graph. The diagrams are a
pleasant time-lapse, but they leave a deeper worry unanswered. The cut property
says the *locally* lightest crossing edge is safe at the current step. But every
pick reshapes the cut for the next step; fresh crossings can appear and old ones
may disappear.

Suppose at some iteration the cut offers a crossing of weight $1$ and another of
weight $3$. Greedy, being greedy, lunges for the $1$. But that choice changes the geometry of $S$, which changes
which edges cross the next cut. What if the cheapest edge available afterward is
suddenly $12$, while the neglected $3$ would have opened the door to a lovely
$4$? Across two steps, $1 + 12 = 13$ looks rather embarrassing next to $3 + 4 =
7$.

This is a Dijkstra kind of mindset looking at every graph algorithm with a path minimization lens.
This kind of thinking is akin to thinking that if she smiled at you once, she must be into
you. This is mathematically dubious and romantically catastrophic.


So the question is not whether the current edge is light. The question is
whether, in grabbing today's bargain, we accidentally stepped on tomorrow's
sale.

A reasonable worry. Why doesn't this look-ahead trap ever snap shut on Prim?


The cut property is stronger than "the lightest crossing edge is locally safe."
It says *there exists* an MST containing the full $A$ and *will continue to
exist* an MST containing $A \cup \{e\}$ once we add the lightest crossing edge
$e$. Each greedy pick comes with a witness MST. Possibly a different witness
from the one we had a step ago; that is fine. What matters is that the witness
exists at every step, the invariant holds, and the algorithm can keep extending.

The proof is the exchange argument. Suppose, for contradiction, that no MST
contains $A \cup \{e\}$. The invariant before our step says *some* MST $T^*$
contains $A$  so $T^*$ exists, but $T^*$ does not contain $e$.

Add $e$ to $T^*$. A spanning tree gains exactly one cycle whenever you add any
non-tree edge, so $T^* \cup \{e\}$ has a unique cycle. That cycle must cross the
cut at least *twice*: $e$ crosses once, and the cycle eventually has to return
to its starting side. Pick any other crossing edge $e'$ on the cycle; $e' \in
T^*$.

Two micro-claims in that paragraph deserve their own pictures. First, *why*
adding a non-tree edge to a spanning tree creates exactly one cycle:

![Why adding a non-tree edge to a spanning tree creates exactly one cycle](/images/tree-cycle-argument.svg)

Panel (1) draws a tree $T$ and picks two non-adjacent vertices $u$ and $v$. Panel (2) highlights the path from $u$ to $v$ inside $T$ and uses a structural fact about trees: between any two vertices, there is *exactly one* path (otherwise two distinct paths would close into a cycle, contradicting acyclicity). Panel (3) adds the non-tree edge $e = (u, v)$. That edge, together with the unique $u\!\to\!v$ path, forms a closed loop; one cycle. 

Second, the half-step the proof slides over: we called $e$ "the lightest crossing edge," not "a non-tree edge of $T^*$." Why is $e$ automatically non-tree with respect to $T^*$?

![Why e is a non-tree edge of T* under the assumption](/images/e-not-in-tstar.svg)

It is the *assumption* doing the work. If $T^*$ happened to contain $e$, then $T^*$ itself would be an MST containing $A \cup \{e\}$ but the very thing we assumed (for contradiction) was that *no* MST contains $A \cup \{e\}$. So under the assumption the only consistent case is $e \notin T^*$. That is what makes $e$ a non-tree edge of $T^*$, and that is what licenses the "add $e$, get one cycle" step that comes next.

Now exchange: $T^{**} = T^* + e - e'$. We deleted one edge of the cycle, so $T^{**}$ is acyclic; we kept connectivity, so $T^{**}$ spans every vertex. $T^{**}$ is a spanning tree. Its weight is
$$
w(T^{**}) = w(T^*) - w(e') + w(e) \le w(T^*),
$$
because $e$ is the *lightest* crossing edge and $e'$ is a (not-necessarily-lightest) crossing edge, so $w(e) \le w(e')$. Since $T^*$ was an MST, $T^{**}$ is an MST too. And $T^{**}$ contains $A \cup \{e\}$ contradicting the assumption that no MST doe.s


Now back to the "$1 + 12$ vs $3 + 4$" trap. If a tree using the heavier-now branch $(3, 4, \ldots)$ were optimal, the exchange argument would surface an at-least-as-light tree containing the locally lighter edge $1$ instead of $3$. The crossing edges $1$ and $3$ both cross the *same* cut at the moment of decision; their cycle is what the argument operates on. The locally lightest crossing edge is not just safe it is *globally* safe in the only sense that matters, namely "an MST containing it exists."

That is the discipline behind Prim. The recipe (pick the lightest crossing edge of $(S,V \setminus S)$) is myopic. The cut property is the proof that one-step myopia is *enough* to reach a global optimum.


### Incremental construction

The cut property gives you an *incremental design*: maintain a
partial $A$, find a cut that respects $A$, take the lightest crossing
edge, add it to $A$, repeat $n - 1$ times. The trivial start: $A = \emptyset$.



### Naive $O(nm)$, and why dense graphs hurt



So here is the algorithm. We iterate $n-1$ times; on each pass we trudge through
every edge $(u,v)$, check whether exactly one endpoint sits in $S$ (the other
exiled to $V \setminus S$), and keep a tab on the lightest such crossing.


```text
S, A = {start}, []
repeat n - 1 times:
    best = ∞
    for every edge (u, v, w) in the graph:
        if (u ∈ S) xor (v ∈ S):
            if w < best:
                best = w
                pick = (u, v, w)
    A.append(pick)
    S.add(the V\S endpoint of pick)

```


The inner for is $O(m)$. The outer loop fires $n-1$ times. Total: $O(n \cdot
m)$. Beautiful. Hang it on the fridge.

Except. A dense graph has up to $\binom{n}{2} \approx n^2/2$ edges, so $m =
\Theta(n^2)$ and our tidy bound blows up to $O(n^3)$. Sparse graphs we'll let
off with a warning.




Also, it is worth briefly time-travelling back to the 1950s. The language we
casually throw around today asymptotic notation, $(\Theta)$, amortized analysis,
all the sacred symbols of theoretical computer science were not standard classroom vocabulary(until {{< mention "Donald Knuth" "https://en.wikipedia.org/wiki/Donald_Knuth" >}} wrote his famous paper {{< mention "Big Omicron and Big Omega and Big Theta" "https://dl.acm.org/doi/pdf/10.1145/1008328.1008329" >}}
). Yet Prim could still *feel* that this approach was inefficient.

_“The important thing isn’t can you
read music, it’s can you hear it.”_ :: {{< mention "Oppenheimer(film)" "https://en.wikipedia.org/wiki/Oppenheimer_(film)" >}} Prim may not have had modern complexity
theory notation laid neatly before him, but he could hear the inefficiency
anyway.


### Speedup attempt #1: `partner[v]` from $S$ and the hooking trap

Okay, so what was very intuitive for Prim lets try to get through it. The
obvious move: avoid rescanning all $m$ edges every iteration. Keep a small array
indexed by $S$,

$$ \texttt{partner}[v] = \arg\min \big\{\, w(v, u) : u \in V \setminus S
\,\big\}, \qquad v \in S. $$

$\texttt{partner}[v]$ is just "$v$'s lightest neighbour on the $V \setminus S$
side of the cut." The global lightest crossing edge is then the minimum over
$|S| \le n$ partner entries  $O(n)$ per iteration, not $O(m)$. That looks like
an immediate $O(n^2)$ total.

![Partner[v] from S  the staleness trap,
animated](/images/partner-from-s-animation.svg)

Until you remember that the partners require upkeep.(Wise words from a 32 year old unmarried _boy_)
After all, every time you ask “what is `partner[a]`'s lightest edge?”, someone has
to actually know the answer and update the answer everytime an edge moves to $S$.

Watch `partner[a]` over the first four picks. The graph: $a$ at the centre, four
spokes $b, c, d, e$ (weights 1, 2, 3, 4) and three leaves $h, g, f$ hanging off
them. Start with $S = \{a\}$.

Iteration 1: pick $(a, b)$. `partner[a]` was $(b, 1)$, now the original `partern[a]` information is stale because $b$
has crossed into $S$ and that edge is internal. Rescan $a$'s remaining $V
\setminus S$-neighbours $\{c, d, e\}$  **3 edge scans**  and reset `partner[a]
= (c, 2)`.

Iteration 2: pick $(a, c)$. `partner[a]` is stale **again**. Rescan $\{d, e\}$ ::
**2 scans**.

Iteration 3: pick $(a, d)$. Stale **again**. Rescan $\{e\}$ :: **1 scan**.

Iteration 4: pick $(a, e)$. Stale **again**. Zero left to scan, but the
bookkeeping has paid 3 + 2 + 1 = 6 redundant edge inspections, all from a
single vertex.

That is the trap. **The same vertex $a$ keeps re-paying because $a$ keeps being
the one whose partner just got consumed.** In the worst case, *every* $v \in S$
has its partner pointing at the moving $u$, so a single move triggers $\sum_{v}
\deg(v)$ rescans. Across $n$ moves the rescans compound *multiplicatively*:
$O(nm)$. We are exactly where we started.

Prof. CPR sometime around this moment said  *"the genius differs from ordinary people
right here."* 

The natural impulse is to put the bookkeeping on the side that
already carries the partial tree. When I first worked on this problem, my
thought process had all the confidence of a Hollywood mathematician. You know
the scene: dramatic music, equations floating in the air, and suddenly I'm the
computational equivalent of John Nash in a bar discovering Nash equilibrium.
*"Of course! The set $S$ keeps growing, all the action is in $S$, so let's store
everything in $S$."*

Which is exactly the sort of idea that feels brilliant for about three minutes.

The catch is that every time you move a vertex (u) across the cut, your carefully
maintained $S$-side bookkeeping doesn't politely update itself.
The data structures you lovingly built are suddenly about as useful as last
semester's lecture notes , which you painstakingly maintained with three colored pens, during a closed-book exam.

### The flip: `partner[v]` from $V \setminus S$ and the cheap update

Now the aha moment. Keep the partner array on the *other* side:

$$
\texttt{partner}[v] = \arg\min \big\{\, w(v, s) : s \in S \,\big\}, \qquad v \in V \setminus S.
$$

partner[$v$] for $v \in V \setminus S$ is "$v$'s lightest neighbour back in $S$." Same data, the entries just live on different vertices.

![Partner[v] from V\S  the flip, animated](/images/partner-from-vs-flip.svg)

Same graph, same start, partner now lives on $V \setminus S$: each not-yet-claimed vertex names its lightest link back into $S$. Before the pick: `partner[b] = (a, 1)`, `partner[c] = (a, 2)`, `partner[d] = (a, 3)`, `partner[e] = (a, 4)`; vertices $f, g, h$ have no $S$-neighbour yet, so `partner[$\cdot$] = inf`.

Pick the global min, $(a, b)$ at weight 1. $b$ joins $S$. Now look at the other partner entries.

`partner[c], partner[d], partner[e]` all still point at $a$ and $a$ is still in $S$. **Green checkmarks all round, zero work.** A pointer into $S$ can never be invalidated by an addition to $S$, only potentially improved. Notice $S$ is monotonically increasing.

The only update is for $b$'s $V \setminus S$-neighbours: ask whether the new edge into $b$ beats their current partner. On this graph $b$ has exactly one such neighbour, $h$. One comparison  $5 < \infty$  and `partner[h] <- (b, 5)`. The whole move's update cost: at most $\deg(b) = 2$ comparisons.

**Flip = additive. Rescan = multiplicative.** The bookkeeping style is what changes the cost. Multiplication happens when you keep paying for the same mistake. In the previous version, every time the cut changed we essentially looked around and said, "I have altered the state of the graph; better re-examine my entire life." A full rescan after every update. $O(N^2)$

The flipped partner array is much less dramatic. When a vertex $u$ joins $S$, we don't revisit the entire graph. We only look at the $V \setminus S$-neighbours of $u$ and ask a single question: "Am I a better partner than the one you currently have?" (Again, wise words ...)

That's the whole update. For each $V \setminus S$-neighbour of $u$, one comparison. Cost per move is $O(\deg(u))$. Across all $n$ moves,

$$
\sum_{u \in V} \deg(u) = 2m,
$$

Through _Handshaking Lemma_.

so total update work is $O(m)$. The min-find cost is $O(|V \setminus S|) \le O(n)$ per iteration, $O(n^2)$ across $n$ iterations.

Total: $\boxed{O(n^2 + m)}$.    


### Why textbooks present the $V \setminus S$ formulation as if from nowhere

> *"It is very unnatural for one to have a thing viewed in such a fashion. The spanning tree is growing in $S$ but you are solving the problem with the $V \setminus S$ view, because that is algorithmically more efficient. Even I was not convinced, when I was teaching it, why we suddenly jump to the other side."*

This is Prof. CPR's pedagogical complaint. The textbook opens with "for each $v \in V \setminus S$, maintain partner[$v$]" no naive $O(nm)$ baseline, no failed $S$-side attempt, no asymmetry argument. The student memorises the definition and the bound, and gets none of the *why*. The $V \setminus S$ choice looks like an arbitrary convention you have to swallow.

It isn't a convention. It is the *consequence* of running into a wall. You design from the $S$ side first because that's where the partial tree lives and where instincts point to. The hooking cost reveals the wall. Only then does the flip become both motivated and inevitable. The story below the polished result is the only thing that makes the polished result memorable.

## What I'm taking from this

1. **Read the source paper.**
2. **Equivalent does not mean equally expensive.** Mathematics only cares that
   two expressions are the same; your CPU cares very much how you got there.
   Writing $(x^2-y^2)$ is mathematically identical to $((x+y)(x-y))$, but the latter
   gets away with a single multiplication instead of two. Same answer, fewer
   expensive operations, and often smaller intermediate values. The trick in
   Prim's algorithm is exactly this sort of reformulation: nothing changes
   mathematically, but switching the bookkeeping from $S$ to $V \setminus S$
   turns a costly computation into a cheap one.
3. The "what cut would you like to start with?" question is itself a creativity
   rehearsal.

## Anecdotes (Q&A)

### Polya, *How to Solve It*, and the tunnel

> *"Don't die without reading How to Solve It by Polya."*

Prof. CPR emphatically urges everyone to read {{< mention "How to Solve It" "https://en.wikipedia.org/wiki/How_to_Solve_It" >}}.

Prof. CPR's told us a story: *two thousand years ago, two tribes living on opposite sides of a
hill decide to meet in the middle by tunnelling inward from both ends. They land
within feet of each other. Centuries later, France and England try the same
trick under the Channel and miss by six feet, requiring substantial
re-engineering. Polya's question is the obvious one: how did the tribes pull
that off?*

How does one even begin to look for patterns like that? Thanks to {{< mention "Sahil Kokare" "https://www.linkedin.com/in/sahil-kokare-464116311/" >}}, who slid a copy of the book into my hands, I have at least graduated from staring at the question to thinking about it which Polya would, I think, count as progress.


### Gauss's scaffolding

{{< mention "Prof. Chandrashekhar Laxminarayan" "https://sites.google.com/view/chandrashekar-lakshminarayanan" >}} brings up Gauss:

> *"No self-respecting architect will leave the scaffolding back."*

Prof. Chandrashekhar's real question was whether hanging the dirty laundry out
is, in fact, bad form for a book and if so, how on earth one convinces the
reader (and, more tediously, the publisher) that the origin story is worth
printing. The fact that he could quote Gauss verbatim was impressive enough; the
answer was also equally impressive.

A textbook has a job. Conciseness *is* the job. You want the filtered product,
the distilled thing, so the reader can climb straight up the final structure
without tripping over the scaffolding poles still lying around. Fair. But the
other school deserves shelf space too - the one that opens not with a grand
entrance, but with the quiet, desperate arrival of Hazari Pal, 
{{< mention "City of Joy by Dominque Lapierre" "https://en.wikipedia.org/wiki/City_of_Joy" >}} a
proud farmer carrying his family's entire existence on his back, stepping out of
the dust of a drought-ravaged Bengal into a chaotic Calcutta before a single
fact is allowed onstage.

Both schools deserve to exist. Currently, only one of them owns the printing
press.


### Mergesort and the dawn of computer science

> *"The day before mergesort, the day after mergesort computer
> science has those two eras."*

Prof. CPR also regaled us with the story of mergesort and how much of the modern
world quietly traces back to John von Neumann. In the 1940s, governments and
laboratories were drowning in records, and the dominant technology of the age
was the punch-card sorting machine. Von Neumann, working in the EDVAC era, wrote
mergesort in 1945 as part of the larger dream of programmable computing, and the
astonishing part is that one of the founding demonstrations for modern computers
was simply this: sorting records faster than dedicated industrial hardware.
Bureaucratic efficiency, accelerated to absurd levels, somehow became the proof
that programmable computers were worth building. So yes, in the same world
where you are reading this blog post (and if you have made it this far, thank
you), where computer science researchers are traded between companies like NBA
stars and acquire celebrity status, we are still very much living in von
Neumann’s world.


### Does AI replace this?

No article these days feels complete without eventually arriving at the ritual
question: *“Will AI replace this?”* Naturally, that question surfaced here too.
Prof. CPR’s response was something like (I am paraphrasing) : *even to ask the right
question to a machine, one needs some understanding of the domain itself. And if
you are dissatisfied with the $k$ alternatives the machine produces, you must
also possess enough judgment to disagree with it intelligently. The final word,
at least for the foreseeable future, still belongs to humans.**

{{< mention "Byomkesh Bakshi" "https://en.wikipedia.org/wiki/Byomkesh_Bakshi" >}}
has a nice quote: the hardest lie to catch is the one
lurking just close enough to the truth.



---

### References & links

- **Prof. C. Pandu Rangan** :: Department of Computer Science and Engineering, Indian Institute of Science, Bangalore
- **Robert Tarjan** :: *Depth-first search and linear graph algorithms*, SIAM J. Computing, 1972 :: the articulation-point characterisation CPR walks through
- **R. C. Prim** :: *Shortest connection networks and some generalizations*, Bell System Technical Journal, 1957 :: the original technical report
- **George Polya** :: *How to Solve It*, 1945
- **John von Neumann** :: *First Draft of a Report on the EDVAC*, 1945; the merge-sort program written for it
- **ACM Turing Award lectures** :: Prof. CPR's recommendation to read the interview that accompanies each award; the genesis of an idea is often more instructive than the idea itself
