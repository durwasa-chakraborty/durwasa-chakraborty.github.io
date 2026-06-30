+++
title = "GADTs in OCaml: The Type-Level Superpower Your Other Languages Wish They Had"
date = 2026-06-26T10:00:00+05:30
draft = false
author = 'durwasa'
math = true
comments = true
tags = ['programming', 'type-systems', 'functional-programming', 'gadts']
+++

![Tree : An algebraic data type](/images/gadt.jpg)

As someone with a standing interest in languages, I find myself perpetually on
the lookout for the features that belong to one language alone. Every language
has its flagship concepts - words that resist translation, and that frequently
cannot be mapped one-to-one onto any other.


  That a language bothers to coin a word for a whole bouquet of emotions tells
  you something about the civilization that did the bothering. It's not that
  popular languages have some magical  algorithm for minting words  
  out of thin air ; every language can staple syllables together. It's that a
  culture only files the paperwork for a word when it has tripped over the same
  idea often enough.

  And finer the vocabulary, the pickier the people. A language will
  happily let "rain" cover everything from a romantic mist to a biblical flood,
  but somewhere a language has a dedicated word for "the specific rain that
  ruins a wedding but is great for the crops." We do the same with the eyes:
  watching, seeing, looking, glancing, staring, ogling ; same visual apparatus, wildly
  different intentions, and a restraining order separating the last two. 


Translating concepts between languages, natural or programming alike, usually works well
enough, but fine nuances inevitably leak out in transit. Though "language" is a
broad term, this truth applies perfectly to code.Recently I have been working
through [Real World OCaml](https://dev.realworldocaml.org/), and the way I learn
a new language is exactly this kind of translation: take a pattern I already
understand, find its simpler cousin in the new language, map one onto the other.
Most of the time it works.

And then occasionally you hit a concept that simply does not have
the same impact, or any rendering at all,  in the language you came from. One of
those is **GADTs**. They live comfortably in OCaml and Haskell, and they are
flatly absent from Java,Python, C++ etc.

Absent does not mean impossible. 

People have written full object-oriented programs in
plain C. See Axel-Tobias Schreiner's classic
[*Object-Oriented Programming with ANSI-C*](https://www.cs.rit.edu/~ats/books/ooc.pdf)
,even though C is marketed as a humble imperative language. So when a language's
brochure says "imperative" or "no GADTs here," remember that definitions are
mostly reassurances; they are not hard limiits on what can be expressed.

Here's the deeper reason I won't say "untranslatable": every programming
  language worth the name is Turing-complete, which is a  way of saying
  they're all secretly the same language wearing different outfits. Anything you
  can compute in Haskell you can compute in C, in Python, Java, Rust, in assembly. None of them
  can do something the others fundamentally can't.


But a language having a feature *natively* gives you a wonderful motivation in
learning it. So: GADTs. 

 

## First, what is an ADT?

An **algebraic data type** is a type built by combining other types with "and"
(products :: a constructor holding several fields) and "or" (sums :: a choice
between constructors). The humble binary tree is a textbook sum-of-products:

```text
            10
           /  \
          6    28
         / \ 
        1   7
```

```ocaml
type 'a btree =
  | Leaf of 'a
  | Branch of 'a * 'a btree * 'a btree
```

```ocaml
let example_tree : int btree =
  Branch (10, Branch (6, Leaf 1, Leaf 7), Leaf 28)
```

A value of this type is *either* a `Leaf` *or* a `Branch` , it's a sum type, an
either-or. Which raises a sneaky question: given a `btree`, do I know whether it
was built with `Leaf` or `Branch`? The
value itself doesn't advertise its constructor on its sleeve.

## The limitation: a value forgets who made it

Let's shrink the example down to something we can hold in one hand, a tiny
expression language:

```ocaml
type expr =
  | Int of int
  | Bool of bool
  | Add of expr * expr
```

Once I'm holding an `expr`, do I know whether I'm an `Int`, a `Bool`, or an
  `Add`? I do not. The constructor has done its job and left. The value doesn't
  remember who created it and
  the created thing has no innate knowledge of its creator. This is a nice predicament for both
  theological studies and compiler design.

  A function cannot have a chaotic return type. The moment I write
  
  ```ocaml
  let rec eval = function ...
  ```

  the compiler demands to know the *exact* type `eval` outputs. And here I'm
  stuck: `eval` needs to return an integer sometimes (for `Add (Int 2, Int 2)`)
  and a boolean other times (for `Bool true`). I cannot return `int | bool` no
  such type exists. So the motivation is **unified packaging**: I invent a
  single type, `value`, that acts as a diplomatic passport for integers and
  booleans alike.

  ```ocaml
  type value =
    | VInt of int
    | VBool of bool
  ```

  But unifying the distinction to please the compiler isn't tax free it just
  defers the bill. We now pay a runtime tax every time we actually compute.
  We can't simply write `x + y`; first we have to unpack the boxes reach
  inside `x` and `y`, confirm each is a `VInt`, pull out the integers, *then*
  add.

  And that unpacking is exactly where it bites. If someone adds a `VInt 4` to a
  `VBool true`, the mismatch isn't caught when the program is *compiled* it's
  caught when the program is *run*, because the only way to tell the two apart
  is to open the box, and opening the box is something that happens during
  evaluation. We pushed the distinction out of the type and into the *data*, so
  the type checker is blind to it; the error has nowhere to surface but
  **runtime**. Which is what a type system is supposed to catch 
  before a single line executes.


Now the evaluator. Functional-programming hygiene says a function should do one
thing, so we hoist the "give me an int or throw an exception" logic into its own helper ;
exactly the move a Java programmer makes when they isolate the cast-or-throw in a
getter and keep the main method tidy:

```ocaml
let get_int = function
  | VInt x -> x
  | _ -> failwith "type_error"

let rec eval = function
  | Add (a, b) ->
      let x = get_int (eval a) in
      let y = get_int (eval b) in
      VInt (x + y)
  | Int a  -> VInt a
  | Bool a -> VBool a
```

This works, for some definition of "works":

```ocaml
# eval (Add (Int 4, Int 3)) ;;
- : value = VInt 7
```

But nothing stops me from building a nonsense expression, and the
type checker waves it right through:

```ocaml
# let bad : expr = Add (Bool true, Int 3) ;;
val bad : expr = Add (Bool true, Int 3)

# eval bad ;;
Exception: Failure "type_error".
```

The `Add (Bool true, Int 3)` builds happily. What we really want is to make
`Add (Bool true, Int 3)` *inadmissible* ; rejected the moment someone tries to write
it down.The dream would be a type that remembers what its evaluation produces something
like an `expr`that remembers  its result type:

An ordinary `'a expr` forces *every* constructor to produce the *same* `'a expr`.
There's no way to say "`Int` makes an `int expr` but `Bool` makes a `bool expr`."
For that, we need the constructors to each declare their own result type.

## Enter GADTs

**GADT** stands for *Generalized Algebraic Data Type*. The "generalized" part is
exactly the freedom we just wished for: each constructor gets to specify the
concrete type it returns. Think of it, for now, as a *refinement* we want to
know which branch we came from, and have the compiler remember it for us.

```ocaml
type _ expr =
  | Int  : int -> int expr
  | Bool : bool -> bool expr
  | Add  : int expr * int expr -> int expr
```

Read the constructor signatures like  type judgements: `Int` takes an `int`
and yields an `int expr`; `Bool` yields a `bool expr`; and `Add` only accepts two
`int expr`s and yields an `int expr`. Each branch now has a *different* return
type, and that difference is the whole game. The compiler can learn the type
parameter from the constructor.

Now the evaluator becomes a bit more cleaner:

```ocaml
let rec eval : type a. a expr -> a = function
  | Int n      -> n
  | Bool b     -> b
  | Add (x, y) -> eval x + eval y
```

No `value` type. No `get_int` and therefore no `failwith`. The function's type is
`'a expr -> 'a`: evaluating an `int expr` gives an `int`, evaluating a
`bool expr` gives a `bool`.

That `type a.` annotation is not decoration; and it's worth
understanding *why* you can't drop it (more on that below). It introduces a
**locally abstract type** and promises the compiler that `eval` is polymorphic in
`a` and may call itself at *different* instantiations of `a`. In the `Int` branch
the compiler learns `a = int`, so returning `n : int` is returning an `a`. In the `Bool` branch
the compiler learns `a = bool`, so returning `n : bool` and so on.

The whole reason we climbed up here:

```ocaml
# let bad = Add (Bool true, Int 3) ;;
Error: This expression has type bool expr
       but an expression was expected of type int expr
       Type bool is not compatible with type int
```

The bad (inadmissible before) expression no longer *compiles*. We traded a runtime `failwith` we
had to remember to write for a compile-time error the language hands us for free.
That is a strictly better place to discover your bug; at your desk, not in
production at 3 a.m with your pager constantly going on.

GADTs are not a cure-all; Real World OCaml has a whole
[Limitations of GADTs](https://dev.realworldocaml.org/gadts.html#limitations-of-gadts)
section, and we'll touch on the sharp edges at the end. But first, the fun part:
watching other languages try (and fail).

> **A Quick Note** This is not a critique of Python or Java. Python is the language I owe my career to. Dismissing a language that powers global infrastructure just to join an elitist bandwagon is shallow and I won't partake in that exercise unless I have actually stretched the language to its limits. Every language makes deliberate trade-offs. Python prioritizes speed of thought, and enormous flexibility over strict compile time policing. (GADTs) live at the opposite end of that spectrum. When Python "can't" do, what follows isn't a failure it is simply a language doing exactly what it was designed to do. This isn't a knock; it's a comparison of different targets.

## Why Python can't!


Let's port the expression language to Python and evaluate `Add(Bool(True), Int(5))`:

```python
class Int:
    def __init__(self, v): self.v = v
class Bool:
    def __init__(self, v): self.v = v
class Add:
    def __init__(self, l, r): self.l, self.r = l, r

def eval(e):
    if isinstance(e, Int):  return e.v
    if isinstance(e, Bool): return e.v
    if isinstance(e, Add):  return eval(e.l) + eval(e.r)

print(eval(Add(Bool(True), Int(5))))
```

In OCaml this expression refused to compile. In Python it doesn't merely run ;
it cheerfully prints:

```text
6
```

**Six.** and not a `TypeError`, not a crash.
Python `bool` *is* a subclass of `int`:

```python
>>> bool.__mro__
(<class 'bool'>, <class 'int'>, <class 'object'>)
>>> isinstance(True, int)
True
>>> True + 5
6
```

`True` is, numerically, `1` (this is fixed by PEP 285, which made `bool` a
subclass of `int` for backward compatibility). `bool` doesn't override addition,
so `True + 5` dispatches straight to `int.__add__` and gives `6`. The "type
error" we *wanted* ; adding a boolean to a number. This is not an error in the Python universe. A wrong answer that runs is worse than a crash: a crash at
least tells you something is broken.

## Why Java can't (and it's instructive)

> The following section's code snippets have been written using claude=opus-4.8. While my experience is primarily with Java 8, the landscape has moved far past it, introducing a vast array of new features and capabilities that I am now exploring and know very little of. 

Java fans will object that modern Java is no new kid in the block; sealed interfaces (Java 17);
pattern matching for `switch` and records (Java 21) give us real ADTs with
*compile-time-exhaustive* matching. Fair. Let's give Java its best shot:

```java
sealed interface Expr<T> permits IntLit, BoolLit, Add {}
record IntLit(int v)                          implements Expr<Integer> {}
record BoolLit(boolean v)                     implements Expr<Boolean> {}
record Add(Expr<Integer> l, Expr<Integer> r)  implements Expr<Integer> {}
```

Credit where due: because `Add` demands two `Expr<Integer>` arguments, the
construction half actually *is* caught at compile time 

```java
var bad = new Add(new BoolLit(true), new IntLit(3));
// error: incompatible types: BoolLit cannot be converted to Expr<Integer>
```

So far, so respectable. The wheels come off at the *eval*. Let's write the
`eval` the honest way, without any casts:

```java
static <T> T eval(Expr<T> e) {
    if (e instanceof IntLit i)  return i.v();   // i.v() is an int; we claim it's T
    ...
}
```

```text
error: incompatible types: int cannot be converted to T
```

There it is. Even though we just *checked at runtime* that `e` is an `IntLit`,
the type system refuses to conclude that `T = Integer`. Java's generics
have no mechanism to narrow the type parameter per constructor; that is exactly
the GADT power Java lacks. Your only way out is to lie to the compiler with an
unchecked cast:

```java
@SuppressWarnings("unchecked")
static <T> T eval(Expr<T> e) {
    if (e instanceof IntLit i)  return (T) Integer.valueOf(i.v());
    if (e instanceof BoolLit b) return (T) Boolean.valueOf(b.v());
    if (e instanceof Add a)     return (T) Integer.valueOf(eval(a.l()) + eval(a.r()));
    throw new IllegalStateException("unreachable");
}
```

This compiles (with a warning). And `eval(new Add(new IntLit(4), new IntLit(3)))`
does print `7`. But "is this compile-time safe?" Not really. That `(T)` cast is a
**promise**, not a proof and because generics are erased. Hand `eval` a deliberately mislabeled expression and the lie comes due at
runtime:

```java
String s = ExprUnsound.<String>eval((Expr<String>)(Object) new IntLit(42));
// Exception in thread "main" java.lang.ClassCastException:
//   class java.lang.Integer cannot be cast to class java.lang.String
```

So Java's story is: discrimination happens at *runtime* (`instanceof`), the
generic parameter `T` is never narrowed, and the bridge from "matched `IntLit`"
to "result is `Integer`" is a cast the compiler accepts on faith.


We saw that Java has no way to tell `List<Integer>` from `List<String>`: erasure
  collapses both into plain `List`. Presumably there is a header that says `List`. 
  So what? Why not bolt on a second header field and
  store which type argument it was created with?

  Two facts have to be kept apart here, because they look like one and aren't:

  Erasure is a runtime fact. It's why the `T` cast can't be verified. There's no
  type information left at runtime to check it against. 
  It's that Java's type checker
  has no mechanism to refine a type parameter inside a branch; to add an equation
  like `T = Integer `and reason with it.

  I'll set the backward-compatibility argument aside; it's mostly politics. The problem
  with the extra header is more fundamental. Consider:

```java
  static <T> List<T> foo(T x) {
    return new ArrayList<T>();
  }
```
  

  What would you store in that header in place of `T`? `T` is itself a type variable. At
  the point the object is created there is no concrete type to record. To fill the
  header for bookkeeping, you'd have to pass the type in at runtime, threaded through
  every call site of foo. That's not "one more header field"; that's a pretty noticable change
  to the calling convention.


  But here's the kicker: even if we had that oracle; a type handed to us at
  every `instanceof`, all it buys is runtime verification. The absence of GADTs would remain, because that's the separate
  compile-time fact: Java's type system still can't narrow `T` in the branch. Reification (as C# uses which means type information is fully preserved during the runtime)
  fixes the runtime lie; it does nothing for compile-time narrowing. 

Two more pieces make GADTs work, and they're where the historical accidents live:

### Hindley-Milner only does let-polymorphism

Standard functional languages, OCaml, Haskell, lean on the Hindley-Milner type
system (HM from here on). HM is the reason you can write whole programs without
ever annotating a type: the compiler is clever enough to infer them for you. But
that cleverness comes with a rule about *where* generic types are allowed to
live, and that rule is the first thing standing between an ordinary ADT and a
GADT.

HM only gives you **let-polymorphism** :: also called rank-1, or prenex,
polymorphism. A function may be generic, but all of its type variables have to be
nailed down up front: chosen the instant the function is called, and held fixed
for the entire duration of that call.

Take the identity function:

```ocaml
let id : 'a. 'a -> 'a = fun x -> x
```

Its type reads $\forall a.\ a \to a$, and notice *where* the $\forall$ sits out in
front, wrapping the whole arrow. Now contrast two readings that look almost
identical on paper but mean very different things:

- $\forall a.\ (a \to a)$ is a **type**. It says: pick a single type for `a`, and
  once you've committed, I'll hand you a function from that type to itself.
- $(\forall a.\ a \to a)$ is a **value**. It says: here is one function that, all by
  itself, swallows *any* type and returns that same type.

Rank-1 only lets the $\forall$ live in the inside. The moment you need it on the
inside, a single argument that *stays* polymorphic while you use it. Watch it break:

```ocaml
let apply_alt (f : 'a -> 'a) : int * string =
  (f 42, f "hello")
```

This doesn't compile. Inside the body, `f 42` forces `a = int`. Then the compiler
reaches `f "hello"`, where `a` would have to be `string`, and it throws a type
error because `f` got pinned to `int -> int` the instant you applied it the
first time. The $\forall$ was hoisted to the front ($\forall a.\ (a \to a)$), so `f`
is monomorphic for the whole call.

The fix is to stuff the $\forall$ into a box the rank-1 system is willing to carry
around :: __a record_:

```ocaml
type poly = { run : 'a. 'a -> 'a }

let apply_alt (f : poly) : int * string =
  (f.run 42, f.run "hello")
```

The field `run` has type $\forall a.\ a \to a$; the polymorphism now lives *inside*
the record. From the outside, `f` is no longer some awkward higher-rank function,
it's just a value of type `poly`, a plain record. A rank-1 compiler will pass a
`poly` around all day as the universal quantifier is
sealed inside the container.

Here's a case where you can actually feel the inference engine give up. Try this
in `utop`:

```ocaml
let rec deep_pair : 'a. int -> 'a -> 'a = fun n x ->
  if n <= 0 then x
  else deep_pair (n - 1) (x, x)
```

Type inference is, at bottom, solving a system of equations. 

Walk through `deep_pair 2 "hi"`:

- `x` is a `string`.
- The recursive call fires  `deep_pair 1 ("hi", "hi")` - so now `x` is
  `string * string`.
- And again `deep_pair 0 (("hi", "hi"), ("hi", "hi"))` - so `x` is
  `(string * string) * (string * string)`.

To type this, the engine would have to solve `T = T * T`. Is `T` a `string`?
No - `string` isn't `string * string`. Is it `string * string`? No, that isn't
`(string * string) * (string * string)`. The type doubles at every level and the
equation never closes. This is **polymorphic recursion**, and HM flatly cannot
infer it. The only reason the snippet above compiles is the explicit `'a.`
annotation: *you*, the human, promised the type the compiler couldn't deduce.
Drop the `'a.` and it stops working.

So why doesn't Java just do the same ; if you can't solve it at compile time,
throw an error and make the programmer spell it out? That *is* almost exactly
Java's philosophy: being explicit. The language was designed so you
could read a method signature and know precisely what goes in and what comes out,
without running a type-inference engine in your head.

But there's a deeper, more structural reason. Java is built on inheritance, and
inheritance is **subtyping** and mixing subtyping with HM-style polymorphism is
genuinely hard. It's worth seeing why.

When HM solves `T = T * T`, it's juggling *equations*, and equations, unlike stakeholders of a pluralist society, are
comparatively easy to unify.  Say
`x` has type `α`, and `x` gets handed to something that adds `1` to it. Addition
takes `int`s, so `α = int`. One clean algebraic substitution, solved instantly.

Subtyping swaps equations for *inequalities*:

```text
α <: Animal
```

Now you can't simply substitute `Animal` for `α` everywhere :: `α` could be any
subtype. Suddenly every variable drags around an upper *and* a lower bound:

```text
l1 <: x <: l2
l1 <: y <: l3
```

and solving a whole system of those constraints is a far nastier problem than
solving plain equalities. That's the tax subtyping levies, and it's a big part of
why Java never went down the GADT road.

### Locally abstract types

The second piece is that `type a.` annotation we kept sprinkling on `eval`. Think
about what a single match branch promises. Inside `| Int n ->`, you are
and will *always* be, in the `int` branch; the compiler has learned `a = int`,
and that fact holds for the entire branch, no exceptions. The evaluator walks the
expression like a tree, and along every edge the type is pinned: this subtree is
an `int expr`.

So what happens if, somewhere deep in that tree, a branch the type checker
expected to be an `int expr` turns out to hold a `bool`? That's the whole point:
**it never gets that far.** All of this is happening at *compile* time. The type
checker walks the forest of subexpressions, and the instant it meets a `bool`
where an `int expr` was promised, it stops and it doesn't evaluate a thing, it just
reports that the types don't line up and refuses to compile.

![Type-checking an expression tree at compile time: the walk halts the moment a bool turns up where an int expr was promised](/images/gadt-compile-time-forest.svg)

## When to reach for them (and a worked idea)

GADTs are still relatively new to my own toolbox, and the honest framing is that I am trying to see if the GADT fits the problem statement.
We reach for GADTs when a
problem has a *specific structure worth teaching the type system*  not for every
sum type with a handful of constructors but then what are those structures. I am trying to figure out.

So the discipline I'm trying to build is: look at a requirement and ask "is this
*really* a plain ADT, or is there an invariant I keep enforcing by hand and
hoping I got right?" If it's the latter, maybe it's a GADT.

Let me motivate this with the smallest real example I could think of: 
a **logger**. I want to print lines like this:

```text
[user_id=42] [status=200]  -> request ok
```

Every log line is a tiny contract: *first* a user id (an `int`), *then* a status
count, *then* a message. And the question is
**who has to remember that contract?** Me, or the compiler?

### The hand-rolled ADT: the contract lives in your head

The obvious first move is a plain ADT. A field is either a required int or an
optional one, and the real data gets *boxed* into a little wrapper so I can shove
everything into one list:

```ocaml
(* the schema: what each position is supposed to be *)
type field = ReqInt of string | OptInt of string

(* the boxed runtime values *)
type value = VInt of int | VNull

exception Bad_log of string

let rec run schema args msg =
  match schema, args with
  | [], [] -> Printf.printf " -> %s\n" msg
  | ReqInt key :: s, VInt v :: a -> Printf.printf "[%s=%d] " key v; run s a msg
  | OptInt key :: s, VInt v :: a -> Printf.printf "[%s=%d] " key v; run s a msg
  | OptInt key :: s, VNull  :: a -> Printf.printf "[%s=null] " key; run s a msg
  | ReqInt key :: _, VNull  :: _ -> raise (Bad_log (key ^ " is required, got null"))
  | _ -> raise (Bad_log "schema and args don't line up")

let () = run [ReqInt "user_id"; OptInt "retries"] [VInt 42; VInt 3] "request ok"
```

This works, for the usual unsatisfying value of "works." But look at everything
*you or the client* just signed up for:

- You have to **box** every value by hand; `VInt 42` instead of a plain `42`.
- You have to keep the schema list and the args list the **same length and in
  the same order**. Nothing stops you from passing two fields and three values.
- The "a required field can't be null" rule is just a branch you *hope* you
  remembered to write.

When any of that slips, the compiler stays silent. You find out at runtime, as a
`Bad_log` exception which is to say, you find out from your pager. The real
contract ends up living in a comment, or worse, a wiki page nobody reads:

```ocaml
(* Note: pass user_id first, then retries. And never pass null for user_id. *)
```

A comment is a promise nobody enforces.

### The GADT: the contract lives in the type

Now the GADT move. The trick and this *is* the whole intuition; is to make the
**type of the format remember the shape of the function you'll have to call.**
Read these three constructors:

```ocaml
type _ log_fmt =
  | Msg    : string -> unit log_fmt
  | Int    : string * 'a log_fmt -> (int -> 'a) log_fmt
  | OptInt : string * 'a log_fmt -> (int option -> 'a) log_fmt

let rec run : type a. a log_fmt -> a = function
  | Msg text -> Printf.printf " -> %s\n" text
  | Int (key, rest) ->
      fun v -> Printf.printf "[%s=%d] " key v; run rest
  | OptInt (key, rest) ->
      fun v ->
        (match v with
         | Some n -> Printf.printf "[%s=%d] " key n
         | None   -> Printf.printf "[%s=null] " key);
        run rest
```

Read the type parameter as a running tally of "what's still left to supply":

- `Msg` ends the chain and gives back a plain `unit log_fmt` - nothing left to
  fill in.
- `Int (key, rest)` says "I need one more `int` than `rest` does," so its type is
  `(int -> 'a) log_fmt`.
- `OptInt` is the same idea, except the next argument is an `int option`.

Stack a few together and the type does the bookkeeping for you:

```ocaml
Int ("user_id", Int ("status", Msg "request ok"))
(* type:  (int -> int -> unit) log_fmt *)
```

And `run` of that value *is* a function of type `int -> int -> unit`. So you
call it with **native, unboxed values**

```ocaml
let () = run (Int ("user_id", Int ("status", Msg "request ok"))) 42 200
(* [user_id=42] [status=200]  -> request ok *)
```

The editor's autocomplete now *knows* the next argument has to be an `int`. Hand
it a string and it won't compile. Forget an argument and it won't compile. The
"user_id first, then status" rule isn't sitting in a comment or an Atlassian document anymore the
compiler is holding it for you.

That's the intuition in one sentence: **a plain ADT stores data; a GADT stores
data *plus a fact about that data's shape*, and the compiler is allowed to read
the fact.**

> **A caveat, since I'm still exploring this pattern.** For a logger this fixed,
> a GADT is honestly overkill you could just write
> `let log_line ~user_id ~status msg = ...`, call it a day, and most of the time
> you should. I'm reaching for it here mostly because it's a genuinely nice thing
> to learn, and because even at this size it buys *one* real thing: it keeps the
> **runner clean**. Every per-field formatting rule lives in the single `run`,
> and each call site stays an ordinary, type-checked function call instead of an
> array of boxed values. 

### Why bother? Because you don't know tomorrow's fields

You'd never reach for any of this to write *one* logger, once a plain
function is fine for that. GADTs start paying rent when you hand a logging
*library* to a whole company, and every team wants a different line. So let's
hire three teams one at a time and watch what each one actually costs us.

**Team A** wants `[user_id=42]` runner with a single `int`. We already have the `Int`
constructor, so the library doesn't change *at all*. Team A just composes what's
there, and the type of the function falls out on its own:

```ocaml
let () = run (Int ("user_id", Msg "request ok")) 42
(* run here has type:  int -> unit *)
```

**Team B** shows up wanting `[endpoint=/checkout] [status=200] [latency=0.043]` —
a `string`, an `int`, and a `float`. `Int` exists, but `string` and `float`
don't. So we, the team responsible for the core logger library, add **two constructors, once**  and one
matching arm each in `run`:

```ocaml
(* in the type *)
| Str   : string * 'a log_fmt -> (string -> 'a) log_fmt
| Float : string * 'a log_fmt -> (float  -> 'a) log_fmt

(* in run *)
| Str   (key, rest) -> fun v -> Printf.printf "[%s=%s] " key v; run rest
| Float (key, rest) -> fun v -> Printf.printf "[%s=%g] " key v; run rest
```

That's the entire change. Now Team B assembles its *own* shape:

```ocaml
let () =
  run (Str ("endpoint", Int ("status", Float ("latency", Msg "served"))))
    "/checkout" 200 0.043
(* run here has type:  string -> int -> float -> unit *)
```

**Team C** arrives wanting `[query=SELECT 1] [success=true]` so the grocery list reads  a `string` and a
`bool`. `Str` is already on the shelf, courtesy of Team B; only `bool` is new, so
the library grows by exactly **one** more constructor (and one more `run` arm):

```ocaml
(* in the type *)
| Bool : string * 'a log_fmt -> (bool -> 'a) log_fmt

(* in run *)
| Bool (key, rest) -> fun v -> Printf.printf "[%s=%b] " key v; run rest
```

```ocaml
let () = run (Str ("query", Bool ("success", Msg "done"))) "SELECT 1" true
(* run here has type:  string -> bool -> unit *)
```

Now look at the *shape of the bill*. The library grew by **one constructor per
new primitive type** `int`, `string`, `float`, `bool`  and that list runs dry
almost immediately; there are only so many primitives. Every team's particular
*combination* of fields cost the library **nothing**, yet each team still got
back a function whose type is checked for *their* exact line, native values and
all.

Price the same three teams on the boxed ADT from earlier and the design takes a reverse constitutional. Every team keeps hand-wrapping
`VInt`/`VStr`, and your runtime `run` has to grow a branch for every fresh
`(field, value)` pairing it might meet. The boilerplate scales with
*combinations*, not types, and a wrong arity or a null as a result of the absence of mindfulness  still lies in wait
to ambush you at runtime at 3 am. The GADT pays one small fixed cost up front so that
"tomorrow's field" is a one-liner instead of another round of pipeline
babysitting at unexepcted hours.

The discipline, restated: The moment you need a value to carry its own context,
or you catch yourself wishing a type knew just a little bit more about its
specific variant; that’s your cue. Don't rely on documentation or runtime checks
to bridge that gap - b  aking those refined values directly into the compiler.

A GADT lets a value carry a little proof about itself
through the type system, so the compiler can recover and *enforce* facts that
other languages relegate to runtime casts, `failwith`s, and hope.
