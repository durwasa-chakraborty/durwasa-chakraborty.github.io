+++ 
title = 'Soundness and Sambar at IISc Bangalore' 
date = 2026-05-07T18:15:04+05:30 
draft = false 
author = 'durwasa' 
math = true 
comments = true 
tags = ['programming', 'lean', 'formal-methods', 'hackathon'] 
+++



A couple of weeks ago I travelled to Bangalore for a hackathon at the **Indian
Institute of Science**, organised by 
{{< mention "Prof. Siddharth Gadgil" "https://math.iisc.ac.in/~gadgil/" >}} and **Emergence AI**. The structure of the hackathon
was: spend the first couple of days getting taught {{< mention "Lean4"
"https://lean-lang.org/" >}} by Prof. Siddharth Gadgil and 
{{< mention "Prof. Prathamesh T. V. H." "https://krea.edu.in/sias/dr-t-v-h-prathamesh/" >}}, then
disappear into a {{< mention "cave for a week" "https://www.youtube.com/watch?v=0SARbwvhupQ" >}} and emerge with a project. The
judges were {{< mention "Anand Rao Tadipatri" "https://dl.acm.org/profile/99661086046" >}} and 
{{< mention "Sidharth Bhat" "https://grosser.science/team/bhat/" >}}; both PhD students in mathematics at
the University of Cambridge.

> **A note on AI assistance.** This post has been run through AI-assisted
> Grammarly and claude-opus-4-7 to make sure the technical details - Lean,
> Plausible, the theorem statement, the build pipeline - are stated correctly.
> The bulk of the prose is the author's own creation. Any wit that lands is
> mine; any that doesn't, well ... unfortunately that is also mine.

## Why this problem, and not a different one

The seed of the idea actually traces back to my time at a previous role, where
I'd seen the shape of this problem before. It's not that the engineering answer
doesn't exist; it's that doing it *properly* is tedious and expensive, and under
shipping pressure those efforts quietly slip down the backlog. A hackathon is
exactly the kind of setting where you can pick up that kind of work, ignore the
*time-to-ship* calculus, and carve out the proof of concept that the production
world doesn't usually have room for.

Concretely, the older version of the problem looked like this. You write a
[MapStruct](https://mapstruct.org/) - or any equivalent annotation-driven mapper
that lines up your JSON keys 1:1 with Java instance variables. The moment a key
points at a *nested object* (`"address": {"id": 1, "city": "Bangalore"}`), you
reach for inheritance and a foreign key, and the nested record becomes its own
table:

```json
{
  "name": "alice",
  "age":  30,
  "address": { "id": 1, "city": "Bangalore" }
}
```

```java
@MappedSuperclass
abstract class Identified {
//  every row gets a primary key
    @Id @GeneratedValue Long id;
}

@Entity
class Address extends Identified {
//  → addresses(id, city)
    String city;
}

@Entity
class User extends Identified {
    String name;
    int age;

    @OneToOne
    @JoinColumn(name = "address_id")
    //  → users(id, name, age, address_id)
    Address address;
    //    FK: address_id → addresses(id)
}
```

Two moves are happening at once. `extends Identified` is the *inheritance*
move - every entity reuses the same primary-key column without restating it.
`@OneToOne` + `@JoinColumn` is the *foreign-key* move the annotation tells the
mapper to flatten the nested JSON object into a separate table.

Once you have the mapping, the pipeline writes itself on a whiteboard:

$$ \texttt{datalake JSON} \;\longrightarrow\; \text{transformation}
\;\longrightarrow\; \text{Parquet wrapper} \;\longrightarrow\;
\text{SQL-queryable DB} $$

(That [Parquet](https://parquet.apache.org/) wrapper in the middle is the part
that lets the SQL side read the columnar bytes.)

And then the punchline: document search doesn't have indexing, SQL does. So the
same logical query - *find users in Bangalore older than 25* - takes a full scan
over the million JSON files but in SQL takes a B-tree lookup. The "why bother
translating" question answers itself the moment your dataset stops fitting in
RAM.

So that's the production-world problem. We took it, cut a slice off it, and
asked: **what does a proof-of-concept of *the translation step itself* look
like - one where we're not just trusting the translator, we're *proving* it?**

## The unlock: cross-language design via proofs

A huge shoutout to {{< mention "Anirudh Gupta"
"https://in.linkedin.com/in/anirudh-gupta-45a52b295" >}}, one of the TAs for the
hackathon whose framing of cross-language design was the unlock for us. He's
still a graduate student, but carries the kind of wisdom that made me
double-check what year he's in. The idea, in his words:

> Once you cleanly separate the **specification** (a Lean-checked proof) from
> the **implementation** (say a python app that consumes it and gets on with the
> building), you stop trying to verify everything and start letting applications
> communicate *through proofs*.

That sentence is the load-bearing wall of the whole project. You don't have to
verify the FastAPI request handler. You don't have to verify the React
component. You just have to verify the *one function in the middle that turns a
jq query into a SQL query*, and ship that as a binary the rest of the system
trusts.
## The theorem

Once we agreed on the shape, {{< mention "Vimala Soundarapandian"
"https://in.linkedin.com/in/vimala-soundarapandian" >}} came up with the theorem
we needed to show. She wrote it on paper, in ink, with the casual confidence of
someone for whom this was obviously the right statement and took me one full day
to comprehend. I
have that paper. I will someday auction it on eBay.

The cast: a JSON database jdb, a SQL database sdb, a jq query jq, a translator
convert, and an evaluator on each side. Each evaluator returns both the
(possibly updated) database and the result of the query:

$$ \begin{aligned} \texttt{convert} \;\;&:\;\; \texttt{jquery} \to
\texttt{sqlquery} \\ \texttt{eval\_jquery} \;\;&:\;\; \texttt{jdb} \to
\texttt{jquery} \to \texttt{jdb} \times \texttt{jres} \\ \texttt{eval\_squery}
\;\;&:\;\; \texttt{sdb} \to \texttt{sqlquery} \to \texttt{sdb} \times
\texttt{sres} \end{aligned} $$

We also need two notions of equivalence - one on databases, one on query
results:

$$ \begin{aligned} \texttt{equiv\_db} \;\;&:\;\; \texttt{jdb} \to \texttt{sdb}
\to \texttt{Prop} \\ \texttt{equiv\_res} \;\;&:\;\; \texttt{jres} \to
\texttt{sres} \to \texttt{Prop} \end{aligned} $$

And the property - the property - is:

$$ \begin{aligned} \forall\, \texttt{jdb}\;\texttt{sdb}\;\texttt{jq}.\quad &
\texttt{equiv\_db}(\texttt{jdb}, \texttt{sdb}) \\ \implies\;& \text{let }
(\texttt{jdb}', \texttt{jres}) = \texttt{eval\_jquery}(\texttt{jdb},
\texttt{jq}) \text{ in} \\ & \text{let } (\texttt{sdb}', \texttt{sres}) =
\texttt{eval\_squery}(\texttt{sdb}, \texttt{convert}(\texttt{jq})) \text{ in} \\
& \texttt{equiv\_db}(\texttt{jdb}', \texttt{sdb}') \;\land\;
\texttt{equiv\_res}(\texttt{jres}, \texttt{sres}) \end{aligned} $$

In plain English: if your two databases held the same data to begin with, then
running the jq query on the JSON side and the translated SQL query on the SQL
side leaves the two databases still holding the same data, and hands back
results that agree too. The translator can't lie. The two views can't silently
disagree - neither in what they store, nor in what they return.


> A brief diplomatic note on the term *jquery*: no, this is not
> [jQuery](https://jquery.com?utm_source=chatgpt.com), the venerable front-end
> library. Here, “jquery” simply means “a query written in jq,” because “jq
> query” sounds like a typographical error and is extremely difficult to pronounce.
> Henceforth, *jquery* shall mean: a query written in jq.


The proof strategy is what you'd expect once you stare at the theorem:
case-split on the JQuery constructor (find / drop / prepend / clear / length /
modify), and discharge each arm with the corresponding lemma about eval_squery.
Six cases, six matching SQL operators, one universally quantified miracle at the
top and let the machine do most of the discharging.

## What the front-end and back-end actually do

There is no long-running Lean server. There is no socket or RPC. The wire protocol between FastAPI and the
Lean kernel is, quite simply, 
`subprocess.run`. 

That sounds primitive. It is, in fact, the whole trick.

Three binaries fall out of `lake build`. Each one is a single-shot RPC:
ask Lean to do the work, hand back a structured JSON object on stdout.

- **`sqlGenMain`**  spawned once *per query*. `argv` is the jq string.
  `stdout` is a JSON blob: `{sql, jq, jq_result, sq_result, match,
  jquery_case, squery_case}`. Timeout 5 s. The `match` field is the
  per-query proof witness - it's `true` iff the kernel's reduction of
  `eval_jquery seedDB jq` and `eval_squery sd (jquery_to_squery jq)` are
  structurally equal. We don't trust python to derive that. Python's job
  is to call the binary, parse the JSON, and put it on screen.

- **`propRunner`**  spawned once per `GET /api/properties`. No argv.
  Stdout is a JSON array, one entry per property in `Properties.lean`.
  Timeout 120 s, because Plausible's loop is
  the bottleneck and we'd rather wait than fake it.

- **`proofTrace`**  does *not* run at request time. It runs once at
  Lean-build time, dumps `proof_trace.json` next to the Lean source, and
  the API just `read_text()`s the file. The reason is: the
  runtime Docker image is a python image without `lake`, and
  `proofTrace` needs `lake env` to set `LEAN_PATH` for
  `Lean.importModules`. So we precompute. The cached snapshot is what
  the *Proofs* tab in the UI is reading when it lists axioms per
  theorem.


## The bit I really wanted to ship: proof communication

There's a thing Plausible does inside the Lean editor - when a property fails,
it pretty-prints the shrunk counter-example into the infoview, and the
experience is genuinely magical the first time you see it. But that magic *stays
in the editor*. It is invisible to anyone running the binary, and very invisible
to anyone visiting your web app.

So one of the things I cared about most was teaching the **browser** to show
this. Not as a screenshot, but as live JSON, fetched on every page load,
rendered as cards with the failing property's name, the bug it caught, and the
minimised input that broke it. If you open the *Property tests* tab in
QueryBridge, what you're looking at is Plausible's output, parsed and displayed
structurally.

The same instinct applied to the *theorems themselves*. Lean's `#print axioms`
will tell you the transitive axiom set a theorem depends on, and that's the
closest thing Lean has to a "trust me, this is real" stamp. We wrote a small
Lean binary (`proofTrace`) that runs the chain

$$ \texttt{Lean.importModules} \;\to\; \texttt{Meta.ppExpr} \;\to\;
\texttt{Lean.collectAxioms} $$

dumps the result as JSON, and the *Proofs* tab in the UI renders one card per
theorem with a chip for every axiom it touches. The headline `query_equiv` rests
on exactly the three Lean foundational axioms (`propext`, `Quot.sound`,
`Classical.choice`) and nothing else - and that fact lives in your browser, not
in a slide deck.

## Gearing up for submission: Docker, one click, and the team name

Submission day is the day everyone discovers their `lake build` takes forty
minutes on a fresh machine. Ours did too. The fix was the standard hackathon
fix: a multi-stage `Dockerfile` that builds the Lean binaries once, then ships
them inside a slim Python image. End-to-end, the reproduction is two lines:

```bash
docker run --rm -p 8000:8000 durwasa/querybridge
# open http://localhost:8000
```

That's it. No `lake update`, no `npm install`, no Python virtualenv. One click
and the product runs.

### About the team name

We were **Sambar & Soundness**.

The name pulled double duty. In formal logic, **soundness** means a system
proves only what is actually true under its intended semantics — no falsehoods,
no sleight of hand. That was precisely the guarantee we were chasing: every SQL
query our translator produced had to faithfully preserve the meaning of its
originating jq query. It was also, quite unmistakably, a nod to our team lead’s
surname — *Soundarapandian* (for those who have a penchant for prefixes).

And then there was **Sambar**: an unapologetically Chennai marker, impossible to
miss. As an unrepentant devotee of alliteration or *अनुप्रास अलंकार* (anupras
alankar) I found the pairing irresistible. It signaled both where we came from
and, perhaps more importantly, it simply had a very good ring to it.


I briefly considered _Madras Monads_ :: a name rooted in my affection for my
almamater IIT Madras, alliteration and _{{< mention "Madraspattinam"
"https://en.wikipedia.org/wiki/History_of_Chennai#Ancient_area_in_South_India"
>}}_ has indigenous and regional origins and not colonial coinage (_madrasa_, a
Muslim scholar's school and _pattinam_, a sea-shore city). But I discarded it
for one strategic reason: if anyone asked me what a _monad_ was, I would likely
have to mutter “_a monad is a monoid in the category of endofunctors_” without
knowing what any of those words mean - and trust me, I have been trying to
figure out what those words mean for some years now, with no luck. The day I do
genuinely understand _monads_ in the spirit of the ritual I will write the
obligatory monad blog post. Today is not that day.

## Closing - and a thank you

A massive thank-you to {{< mention "Vipul Sharma" "https://vipul.xyz" >}} for
hosting me in Bangalore and personally walking me through the Indiranagar food
circuit because every recommendation landed. And to Dr. Sudeeksha Purushottam, 
who ordered the best pizza in Bangalore while
watching RCB specialize once again in the art of narrowly losing.

To the IISc faculty - Prof. Gadgil and KREA faculty Prof. Prathamesh - for the Lean
tutorials. To Anand Rao and Sidharth Bhat for the feedback.
To Anirudh, our TA, for the framing that ended up shaping the whole project. And to
my senior - Vimala for the theorem, and the rest of *Soundness & Sambar* for one
of the more enjoyable weeks I've spent in front of a terminal.

And lastly, not as a matter of politeness or obligation but because I genuinely
mean it, my sincerest thanks to {{< mention "Prof. KC" "https://kcsrk.info/" >}}
and {{< mention "Prof. Kartik Nagar" "https://kartiknagar.github.io/" >}} , for
their patience. They have been taking on the formidable task of teaching me to
see systems not merely as opaque blobs of code, but as mathematical objects that
can be reasoned about, specified, and verified. Despite my persistent instinct
to default to implementation before intuition, they have continued, with
perseverance, to push me toward a deeper systems-level understanding of
verification. The fact that they have not yet given up on me remains deeply
appreciated.

 
If you’d like to explore what I built, here’s my project:
[QueryBridge](https://github.com/durwasa-chakraborty/QueryBridge). Feel free to
try it out, open an issue, suggest improvements, or simply poke around the code.
Contributions, critiques, and curiosity are all welcome.

