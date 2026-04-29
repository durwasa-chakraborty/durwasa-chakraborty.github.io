+++
title = 'Major Mode El'
date = 2024-01-06T18:08:54+05:30
author = 'durwasa'
tags = ['programming']
+++


>Any sufficiently complicated C or Fortran program contains an ad hoc, informally-specified, bug-ridden, slow implementation of half of Common Lisp.


Before you think I'm crowning Lisp as God's own language after just one blog stint at a coding exercise, hold your horses. I'm not here to bash Object-Oriented Programming or its design patterns. In fact, I believe it's crucial to know these patterns inside out. Only then can you play the game of 'Design Pattern or Anti-Pattern?' with any confidence. Remember, these musings are all from my little corner of the world and don't reflect the hard work of other developers who’ve been sweating over [StarPlat](https://arxiv.org/pdf/2305.03317.pdf).

Now, it's not every day that I wake up with a burning desire to write a major mode for Emacs. Most programming languages already have their major modes written by far more ambitious souls. However, in my journey at IIT Madras I stumbled upon a DSL called [StarPlat](https://arxiv.org/pdf/2305.03317.pdf), which didn't have its own major mode in Emacs. So, I thought, 'Why not?' and set out to create one, partly for the challenge, partly for my own learning.

The syntax highlighting looks something like this:
![syntax-highlight](/images/syntax_highlighting.png)

* Syntax Highlighting:
In Emacs Lisp, we use font-lock-defaults, a buffer-local variable. It needs a list that describes the syntax of the language for which the major mode is set. Here's how it looks for StarPlat:

```lisp

(defvar starplat-font-lock-keywords
  (let* (
         ;; Define the list of keywords
         (keywords '("if" "else" "return" "function" "for" "while" "in"))
         ;; Define the list of built-in-functions
         (built-in-funcs '("iterateInBFS" "iterateInReverse"))
         (data-types '("int" "float" "string" "bool" "SetN" "SetE"))

         ;; accumulate regexp strings for each keyword category
         (keywords-regexp (regexp-opt keywords 'words))
         (data-types-regexp (regexp-opt data-types 'words))
         (built-in-funcs-regexp (regexp-opt built-in-funcs 'words)))
    `(

      (,keywords-regexp . font-lock-keyword-face)
      (,data-types-regexp . font-lock-type-face)
      (,built-in-funcs-regexp . font-lock-builtin-face)
      ;; TODO: add further patterns that require syntax highlighting
         )))
```

Reference: [various syntax highlighting options](https://www.gnu.org/software/emacs/manual/html_node/elisp/Faces-for-Font-Lock.html)



Let's breakdown what is happening here - 

In Emacs Lisp, a pair of values is often represented as a `cons` cell. Here, `(,keywords-regexp . font-lock-keyword-face)` is a `cons` cell where `,keywords-regexp` is the `car` (the first element) and `font-lock-keyword-face` is the `cdr` (the second element).

In detail:

- **`font-lock-keyword-face`**: This is one of the predefined faces in Emacs for syntax highlighting (font-locking in Emacs terminology). Emacs uses it by default to highlight keywords.

- **`,keywords-regexp`**: The comma character at the beginning signals that `keywords-regexp` is a symbol that needs to be evaluated. The rules of `keywords-regexp` have been neatly highlighted in the `let*` block. 

```ascii
         ┌─────────────┐
         │             │
         │ Bag of Words│
         │             │
         │             │
         └──────┬──────┘
                │
  ┌─────────────┼─────────────┐
  │             │             │  Regex Match
┌─┴─┐         ┌─┴─┐         ┌─┴─┐
│   │         │   │         │   │
│Key-         │Built-in     │Data-
│words        │  types      │types
│   │         │   │         │   │
└─┬─┘         └─┬─┘         └─┬─┘
  │             │             │  Transformation 
  └─────────────┼─────────────┘
                │
         ┌──────┴──────┐
         │             │
         │ Bag of Words│
         │             │
         └─────────────┘

```

This particular list, can be a list of keywords, data-types, built-in-funcs
Reference: [extensive documentation](https://www.gnu.org/software/emacs/manual/html_node/elisp/Font-Lock-Basics.html#Font-Lock-Basics) on how to use `font-lock-defaults` and related functions.

Imagine you're teaching a black box to play 'color by numbers' with words. You need to:

    a. Define the pattern (the number).
    b. Assign a color to each pattern.
    c. Watch as the black box colors your words accordingly.

This is essentially the famous 'Interpreter Pattern', but in Lisp, it feels like you're just casually tossing words into a magic cauldron and watching them transform. It's a smooth, almost effortless approach to syntax highlighting that showcases Lisp’s elegance and power.
