+++
title = 'Language Semantics'
date = 2023-03-24T11:07:52+05:30
draft = false
author = 'durwasa'
tags = ['programming', 'essays', 'language']
+++

**Happy Birthday to Me: Musings on Language and Code**

Today, I find myself celebrating my birthday in the tranquil confines of a cozy Airbnb in Vancouver. It’s a momentary escape from the pressures of work, a rare breather amidst the ever-present demands of the office. Through the frost-touched window, I watch the Canadian flag battle the icy winds. Below, I can hear the faint murmurs of a French-speaking couple, my neighbors. A little later, the cadence of Mandarin reaches my ears, no doubt my landlord going about his day. And then, the phone rings. It's my mother, calling from home, so naturally, the conversation begins in Bengali. When my father takes the phone, we switch seamlessly to Hindi. After the call, I set the phone down, open my laptop, and begin composing an email in English. 

As the email closes, I dive into writing code, shifting once again—but this time into the realm of Java. Before long, I realize the data needs some cleaning up, so Python it is. A little infrastructure work follows, and suddenly I’m typing away in TypeScript. 

And so I ask myself: why so many languages? Why do we burden ourselves with this linguistic and computational multiplicity? The world has long sought a universal tongue. Philosophers and scientists alike have proposed solutions, from the constructed language of Esperanto to the formalized structure of Loglan. Yet here we remain, in a world where diversity of language reigns, and we must transition from one to another at a moment's notice.

This rumination takes me back to a particularly memorable lesson from my programming languages (PL) class, where my professor once remarked, "Semantics are greater than syntax." Semantics, the meaning behind the words and symbols, form the bedrock of all communication, be it in human languages or in code. Syntax, the structure and form, is what gives language its recognizable shape, but it is the semantics that give it life. 

Let’s stretch this notion beyond programming and into everyday speech. Take Latin, for example: an ancient language from which so many modern tongues have evolved. For nearly every Latin term, there exists an English counterpart. *Suo Moto* becomes “on its own motion,” *Ad Hominem* transforms into “to the person.” Each word, though different in form, carries with it the same weight of meaning—the same semantics. The symbols may change, but the message remains. 

Yet, in the legal world, Latin has emerged as the lingua franca, a curious decision for a modern field. One might argue that Latin’s dominance stems from its ancient roots, with much of Western legal tradition borrowing heavily from Roman law, making Latin a natural choice. However, this reasoning feels like a mere case of "following the herd" or "going with the flow"—a tendency to adopt something just because it has been done before. There is a deeper explanation, I believe, rooted not in convenience but in the allure of the precision Latin offers. 

Consider Japanese, a language rich with words that have permeated the world of business and management. It's not as if other languages lack equivalents for terms like *Ikigai* (reason for being) or *Kaizen* (continuous improvement), or even *Shoshin* (beginner's mind) and *Gemba* (the actual place). These words evoke a sense of discipline and mindfulness that transcends their simple translations. English may offer approximations, but the adoption of these Japanese terms in global contexts speaks to something greater than just a linguistic gap. It's about the cultural weight behind the words—the ethos that underpins their meaning. 

The same phenomenon occurs when I turn to the world of emotions and the arts. A sad poem can be written in English or Hindi, but when heartache strikes deeply, I turn to the lyrical beauty of Bengali or Urdu / Hindustani for solace. The tender phrases, the soft melancholy of Urdu literature, seem to resonate with the human condition in a way that no mere translation can capture. Personally, I find myself drawn to Tamil songs when in need of comfort, though I must often search for their English translations. And Bengali is a complete universe in its own. I just sometimes feel lucky to be born in a bengali family, to know enough bengali to not search meanings of the word by word but rather need to look up for one word in a sentence. (Ya, its bad but more manageable than say Spanish or Tamil). Even with the meaning intact, the experience feels incomplete; the original language carries a certain emotional depth that translations simply cannot map one-to-one.

This, I think, perfectly illustrates the difference between syntax and semantics. You can change the words, but the true meaning—the emotional or cultural resonance—may not always follow. 



```plaintext
// In C
    for (int i = 0; i < size; i++) {
        printf("%d\n", list[i]);
    }

```

```plaintext
// In C++ 
    for (int val : vec) {
        std::cout << val << std::endl;
    }

```

```plaintext
// In Scala
    // Using foreach to traverse
    list.foreach(println)

    // Alternatively, you can use a simple for loop
    for (elem <- list) {
      println(elem)
    }
```

```plaintext
// In Racket
(define (my-functor f l)
  (map f l))

(my-functor (lambda (x) (displayln x)) list)
```

```plaintext
// In Brainf*ck
+[->,----------]     ; Reads input until 0 (input ended)

>++++++++++++[->+++++++++++++<]>+++.  ; Output first value (65 -> A)
```

Programming languages exhibit a vast range of syntactic and semantic differences, and these differences play a crucial role in how various constructs, such as list traversal, are executed. This variance raises essential questions about the orthogonality of programming languages—specifically, their capacity to support multiple paradigms for achieving similar tasks. For instance, does a language restrict itself to traditional constructs like the `for` loop, or does it embrace functional programming paradigms, offering operations like `map`, `list` comprehensions, or `functors`?

The primary objective in list traversal remains constant across languages, yet the choice of method alters the semantics of the operation. A for loop is often more familiar to many programmers, providing clarity and ease of understanding, while functional approaches, such as a map function with a lambda expression, offer a more elegant and concise method. These functional paradigms tend to generate more robust, maintainable code. Despite both methods being syntactically valid, the choice between them has broader implications for readability, flexibility, and a language’s capacity to facilitate more complex, functional patterns.

This brings us to an important realization: the popularity of a programming language often does not hinge solely on its syntax. A language's success is often driven by its semantics—the underlying meaning behind its constructs—and the ecosystem it fosters, including libraries, frameworks, and tools. Semantics, in this context, are far more critical than simply offering numerous syntactic options. The strength of a language lies in how clearly and efficiently it can express complex ideas, rather than how many ways it allows one to express the same task.


At the heart of this discussion is a critical question: Is it possible to design a language where the syntax is intuitive, yet the underlying semantics are versatile enough to accommodate diverse use cases? Can we create a programming language that is not merely a tool but a medium—one that is expressive, relatable, and capable of encapsulating the nuances of the problems it aims to solve?

The challenge in language design lies in achieving a balance between generality and specificity. A language designed for broad applicability needs to offer both flexible syntax and deep semantic richness, whereas a domain-specific language might prioritize strict syntactic rules to ensure precision and correctness. The ultimate goal is to find a middle ground—a language that offers enough expressive power and robust semantics to handle complex problems while remaining accessible and intuitive to a wide range of developers.
