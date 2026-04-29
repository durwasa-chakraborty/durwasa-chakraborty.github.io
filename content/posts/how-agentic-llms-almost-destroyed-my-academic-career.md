+++
title = 'How Agentic LLMs ~Almost~ Destroyed My Academic Career'
date = 2025-10-19T02:54:45+05:30
draft = false
author = 'durwasa'
tags = ['programming', 'ai', 'essays', 'academia']
+++

> "With great power comes great responsibility."  
> Voltaire (and also every Spider-Man movie ever)

There's this **[video](https://youtu.be/Htmcj6HBN7k?si=qK5Ftruvb0okRagY)** of Chad Smith, the drummer from Red Hot Chili Peppers. He's hearing a song for the first time, no prep, no notes, no second take. And yet somehow, he just gets it. He catches the groove like it's muscle memory, then makes the whole thing sound better.That's the magic of practice. Not the kind where you count hours, but the kind where you repeat something so many times it becomes your second nature, your reflex. Whether it's drumming, coding, or explaining your PhD topic to your relatives without crying, the idea's the same: do it till it's boring, and then keep doing it till it's beautiful.

-------------------------------------------------------------------------------

## Why I'm Writing This
*"Give it away, now…."*

This isn't a psychiatrist's reflection. Not a neuroscientist's analysis. And definitely not one of those "10 ways to master your mind" Medium posts written by someone who just discovered cold showers.
Sure, don't air your dirty laundry in public  but cataloging is how self-reflection begins.
This is just me, a guy who left the industry after five years, thinking he was returning to academia armed with experience, purpose, and maybe even principles. But somewhere along the way, got distracted, mistook hype for progress, and forgot what he actually came here for: to become the best damn teacher of computer science he could be. I took the easy path when I should’ve been struggling and that’s on me.

Agentic LLMs aren't just powerful, they're seductive. They don't simply accelerate your work; they make you feel smarter while quietly stealing the struggle that actually teaches you something. It's like outsourcing your gym reps and still expecting Greek God like phyisque.
If you want the formal takes, you can read these:
- [arXiv:2506.08872](https://arxiv.org/abs/2506.08872)
- [MDPI: LLMs and Knowledge Work](https://www.mdpi.com/2075-4698/15/1/6)

What follows is my own messy field report, a note from someone relearning that reflection only counts when you show up and keep trying.
I get the importance of LLMs, especially in industry, where speed means promotion and profit. So surely, build the next generation of __regex(*)__. But this is about LLMs and the academic world, where the goal isn't just to build faster, but get to level where certain CS skills feel second nature, just like the Chad Smith video.

-------------------------------------------------------------------------------

## Back When Failing Was Learning

*"Scar tissues that I wish you saw,"* 

About eleven years ago, I picked up coding. Like every other kid who bombed a competitive exam, I thought: Fine, I'll redeem myself in competitive programming.
Here's a fossil from that era:  
* [feb_chefeq.c](https://github.com/durwasa-chakraborty/codechef/blob/master/feb_chefeq.c)

If you open it, you'll see spaghetti code, random binaries (don't ask me why I pushed those). I was young and dumb. (Now I'm just not young.)
But here's the thing ;  even in that chaos, I was learning. Every WA, every TLE, every embarrassing commit left a small scar that made the lesson stick.  



Take this snippet for example:
```c
int compare(void const *p, void const *q) {
    return (*(int*)p - *(int*)q);
}
```

There’s no universe in which I could’ve written that myself and that’s fine.
Back then, I was basically a raccoon rummaging through Stack Overflow, stealing shiny bits of code and gluing them together with misplaced confidence. And somehow, that counted as victory.

Truth be told, I was terrible at competitive programming. Infact I landed my first Software Engineer job, not through brilliance, but through sheer repetition and stubbornness of trying and retrying leetcode questions.

Even now, I’d probably sit comfortably in the bottom 10 percent of the CP world.
But you know what? That’s okay. Because, in the hindsight the point was never to be good, better or best,  it was to keep trying long enough.

-------------------------------------------------------------------------------

## Enter the Agentic LLM (a.k.a. The Drug Phase)
*"I got a bad disease, out from my brain is where I bleed…"*

Fast forward a decade. Tools evolved.  
Calculators are tools. Google is a tool. Stack Overflow is a tool.
Agentic LLMs? They're not tools. They're drugs.
They don't just help you; they replace you.  
And the dangerous part? They make you feel like you've understood something when all you've done is follow instructions from a very confident autocomplete.
Take AVL trees. I asked an LLM to write one for me. The cycle went like this:
1. Ask for code.
2. Paste errors back into chat.
3. Repeat.
4. After 2-3 iterations, success! It compiles!

What did I learn? Absolutely nothing about AVL trees.  
But I did become frighteningly good at copy-pasting error messages.
That's the trap.  
LLMs can drag you to the finish line but when the next race starts, you're limping.

-------------------------------------------------------------------------------

## The Real Cost of "Efficiency"
*"This life is more than just a read-through…"*

In industry, the higher you climb, the shorter your onboarding.  
At some point, you're expected to start shipping code the moment your setup script finishes running.
That's only possible if you've built intuition, that Chad-Smith-level rhythm where your hands know what to do before your brain catches up.  
And intuition only comes from pain. Academic exercises are the gym. Skip them, and you'll pull a muscle the day you lift something real.

----

## When the Oracle Lies

*"Psychic spies from China try to steal your mind’s elation…"*

Agentic LLMs are trained on data and data has boundaries.  
So if you're in a course that uses an obscure language or explores theory too new or too weird, the oracle goes silent.
Sure, the Greeks said the Oracle got things wrong because of human arrogance.  
But it's still arrogance when we assume we know enough to let the machine think for us.
I once asked an LLM to generate an automated proof. It gave me a perfect-looking answer ; that didn't prove anything.  
When it failed, I broke the proof into subgoals and brute-forced my way through.
Hours later, I had a "working" proof. But I'd learned nothing about proving.  
I'd just learned to outsource persistence.
It's like thinking you've mastered chess because Stockfish told you which move to play.  
You win the game  but you don't understand why you won.

-------------------------------------------------------------------------------

## The Range Analysis Fiasco

*"Throw away your television, make a break for better days…"*

This semester, I tried it again. There was an assignment on range analysis.  
I thought, why not let an LLM write Kildall's algorithm for me?
It produced beautiful code better formatted than anything I'd write at 2 AM  so I used it.  
Got a polite wrist-slap from the instructor.
Lesson learned? None about Kildall.  
But I did learn that shortcuts come with receipts.
The right move would've been to write it myself, submit, get my points, and then compare my approach with the machine's.  
Post-truth, not a priori.

-------------------------------------------------------------------------------

## The Punchline
*"The more I see, the less I know, the more I’d like to let it go…"*

So yeah, LLMs can accelerate your output. But growth doesn’t come from output, it comes from ownership. Every WA and TLE used to sting because they were mine. My mistakes, my blind spots, my late-night bugs that taught me patience and precision. Every hallucinated proof now feels like a thief in disguise, wearing my tone, borrowing my logic, but stealing the struggle that once made me sharper.

The truth is, I outsourced my curiosity. I let convenience masquerade as competence. Each time I asked the machine for an answer I could’ve wrestled with, I handed over a small piece of my craft. The worst part? It felt good. Efficient. Impressive even. Until one day I realized I hadn’t learned, I had only produced.

Use the machine, but don’t let it use you. Because back then, failing meant learning. Now, if I’m not careful, learning just means asking better prompts and that’s not growth, that’s surrender dressed as progress.

-------------------------------------------------------------------------------

## Why This Is Dangerous

*"Destruction leads to a very rough road, but it also breeds creation…"*

I still remember the first time I came across the phrase “segment trees with lazy propagation.”
It sounded like wizardry, the kind of thing only people who truly got algorithms could whisper about with authority. Even today, I can talk about it, maybe even recognize its pattern and say, “ah, this problem smells like a segment tree.” But if you hand me a keyboard and ask me to implement it from scratch, I’ll almost certainly fumble.

And that’s okay. Because learning something complex was supposed to hurt a little, it demanded time, rigor, and a healthy dose of failure. That discomfort was the tuition fee for understanding.

But with LLMs, you skip that pain. You can conjure a working implementation in seconds  and it feels like mastery, but it isn’t. You haven’t built the muscle; you’ve borrowed it.

-------------------------------------------------------------------------------

## The Self-Roast

*"I’m an ocean in your bedroom, make you feel warm…"*

Once upon a time, I used modus ponens in casual conversation so often that my friends actually nicknamed me Modus Ponens. I thought it made me sound profound. Spoiler: it didn’t. It just made me unbearable.

And here’s the cruel irony. In a recent meeting with a professor, I had the perfect chance to use my redemption arc :: "by Modus Ponens", ... and I blanked. Completely. Months of outsourcing reasoning to machines had dulled my instincts so badly that even modus ponens, my old party trick, didn’t come naturally anymore.

This from someone who once represented his school in the CBSE Math Olympiad. Of course, in my current institute, where every other kid has an IMO gold medal or a unary rank, that’s hardly impressive, its not even worth mentioning. But that’s not the point. The point is: I’m going backwards.

This isn’t self-pity; it’s confession disguised as reflection. A plea for academic sincerity.
Back in school, we mocked rote learning ,memorizing proofs, formulas, steps, calling it mechanical. But maybe there was hidden virtue in that repetition. Because repetition built recall, and recall built intuition. And you can’t reason if you can’t remember. Understanding isn’t just clarity; it’s also memory.

Of course, there’s a sociological layer to this too: you can’t stay decent in indecent times - __Harvey Dent__. When everyone around you is using an LLM to ace their assignments or polish their research, it’s almost naïve not to. After all, the system rewards speed, not struggle. Companies care more about CGPA than craftsmanship.

But pause. Breathe. You might just be tumbling down a slope that looks like progress but isn’t.
That’s when it hit me: I hadn’t just used the machine, I had become dependent on it.

LLMs can generate code, but they can’t give you intuition. The code lives “out there,” not in you. After 112 retries, the machine will eventually cough up something that compiles, maybe even pull it from some forgotten corner of the internet. But those 112 failures should’ve been mine. Because if they were, the next time, it would take fewer. That’s how learning works , it amortizes, it compounds.

Output is not insight.
And that’s the tragedy. I wanted to sound like a logician, but somewhere along the way, I turned into a copy-paste monkey in wannabe-academic robes.

-------------------------------------------------------------------------------

## On PhDs and Integrity

*"Dream of Californication…"*

I often wonder if I’m really cut out for this PhD path.
And honestly, the answer shouldn’t depend on brilliance or hard work, both are overrated currencies. The only question worth asking is: Can I be academically sincere?

Because patterns fade, passion burns out, but integrity, that’s the quiet engine that keeps running when everything else falls apart. If I choose this path, make sure it rests on that foundation. Not on borrowed brilliance. Not on synthetic understanding. But on integrity  the kind that doesn’t need applause to persist.

Maybe six months from now, I’ll be staring down a pile of failures. Maybe I’ll quit, and someday reread these words, a message from a younger version of myself who still believed he could do better. And yet, even then, I hope I’ll remember the rain in the first week of December.
The way it falls in Chennai, like a conversation that refuses to end.
How, somewhere between Marina’s waves and Besant’s cafés, life slows just enough for you to notice the salt on your skin, the weight of the evening, the small ache of wanting something unnamed.

And if I do stay, if I keep walking this road, I hope I remember why. To learn sincerely, to rebuild slowly. Continuing in this path hopefully I mend my ways before it’s too late.

-------------------------------------------------------------------------------

## A Cautionary Note

*"All around the world, we can make time…"*

Computer Science isn’t just a career track anymore. It’s a responsibility.
We’re not merely coding, we’re shaping the language of tomorrow’s machines, and maybe even tomorrow’s minds.

But if our foundations are built on hallucinated proofs and shortcut code, we’re no longer engineers. We’re gamblers, praying the machine rolls a lucky seven.

So here’s my parting note to myself and to anyone still reading : 


### __Don’t win the assignment and lose the career.__
