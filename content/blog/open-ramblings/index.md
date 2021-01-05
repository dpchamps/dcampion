---
title: Ramblings for the New Year
date: "2021-01-04T02:08:49.260Z"
description: ""
categories: [rambles]
comments: true
draft: true
---

## effectsjs, programming language research and what next.

### A new blog, a new year.

A little over a year ago algebraic effects were _hot_.

It was probably Dan Abramov's article[^1] that first caught my
eye, followed shortly by Sam Galton's article [^2]. Yassine Elouafi's
series [^3] was another great source of inspiration.

All the while, Algebraic Effects had also been growing
quite popular on the [r/ProgrammingLanguages](https://www.reddit.com/r/ProgrammingLanguages/) subreddit. 
There week or so where I would read a comment referencing them pretty much daily. The people on `r/ProgrammingLanguages`
are truly awesome. It is hands down one of the best communities on reddit.  I was introduced to
[eff-lang](https://www.eff-lang.org/) through the community. I started hacking in Scheme.

### Digressing, Generators and Control Flow

Completely unrelated, I'd also been really into generators around
that time. This interest had been sparked from two different sources: the
wonderful [standard library abstractions](https://doc.rust-lang.org/std/iter/trait.Iterator.html) that Rust provides, and a
rediscovery of Kris Kowal's GTOR [^4]. Specifically, I intrigued by the usefulness
generators provide to asynchronous abstractions.

In general, I wanted to understand problems related to asynchronous control flow more completely.
There was something that just didn't feel right about the ECMAScript `async/await` specification.
I was unhappy with the way functions composed , and reflecting on several of the codebases that I'd been working on. 
Reasoning about async code is a big pain point that I've observed in the wild. Smart people get it wrong and experienced 
devs can get tripped up.

We've seen a few incarnations
of async control flow, and subsequent best practices. 
ECMAThe need for some kind of sugar to write asynchronous code in a
direct-style is very real. The introduction of
[ES6 Harmony Generator Specification](http://wiki.ecmascript.org/doku.php?id=harmony:generators)
spawned several projects to take advantage of the power generators unlocked (co
[^5] and [^6] suspend for a few). We don't talk about generators that much as a community anymore, 
but I find them fascinating and woefully underused. They are arguably the most powerful primitive
introduced to the language.


There are some open questions that I have, in relation to the async idioms and data structures we 
have in JS. I think that we deserve something better than `async/await` for writing direct-style asynchronous code. 
There are better options, but can we ever have them in ECMAScript.

This question is still on my mind. I _suspect_ that we could have something better.

### effectsjs

Shortly after reading Sam Galton's article, I was fully on board with algebraic
effects and wanted them in ECMAScript. I knew that there was overall two
major approaches to creating an effects system in JS: `throw/catch`
semantics, and generators. After reading a few papers, the
`throw/catch` approach seemed inadequate due to that high level of abuse
required for hijacking the call stack. Generators seemed much more
appealing. My gut told me the transforms for generators would be more natural and more appropriate given that they exist
to suspend and resume stackframes. I'm still confident that generators are the correct
choice for building an effects system in javascript.

### Programming Language Research

I've always _flirted_ with programming language design, but never really committed. Programming language
_research_ really hadn't been on my radar up until this point. At the time I probably would've explained it 
away as an esoteric academic pursuit. I never really considered it relevant to other things that I enjoyed.

However, I decided that I wanted to build an effects system and draft ECMAScript specification to provide some sugar for
the mess of generator code that I was sure would follow. I had this general
inclination that being able to write non-color-changing direct-style
asynchronous programming, would be a game-changer. But in order
to test that hypothesis, I'd need to be able to write some code. And in order to
do that, I'd need to draft a specification and write some
transformations to a modified Babel fork.

Thus, [effectsjs](https://github.com/effectsjs/effectsjs) was born and I
accidentally stumbled into doing amateur programming language research.

### Project woes

The biggest challenge presented itself with
[this issue](https://github.com/effectsjs/effectsjs/issues/19). It is the first
indication I think that something wasn't quite right with the first iteration of
the project. The short of it is that for something written in continuation
passing style (CPS for short), the continuation getting passed _can not_ not
perform without some state and being converted into a Promise. This problem is far-reaching.

It of course has nothing to with functional interfaces, but with preserving the
correct context as control is ceded to the callee such that the virtual stack
remains whole. At it's heart, this is a conflict of two different control-flow
styles. Further exacerbating the issue is that CPS-style code expects
_continuations_, not generators or promises. You can't just perform willy-nilly and 
expect seamless interop with other librarys.

[So EffectsBoundary was introduced](https://github.com/effectsjs/effectsjs/pull/33)
to sort of patch over these problems. However, the boundary solve never really
sat well with me. It's not very elegant, requires too much buy-in and syntax
to do normal things. Worst of all, it requires the consumer to be acutely aware
of the effects-system. Effects are no longer this wonderful sugary prose that
allow you to isolate dependencies and mutations in a meaningful way. Instead,
the `perform` keyword becomes a burden. One must apply mental gymnastics in
order to use it correctly. "Do I need to use an `EffectsBoundary` here?", "Where
is the virtual stack right now?" and so on...

Having said all of that, the effects boundary was not all bad. It actually solved the problem in a
meaningful way in the short term. It would've been totally possible to go on
experimenting with the `EffectsBoundary` in place. But it's not good enough, and
it killed my interest in the project for a while.

### effectsjs Revival

A QRD of the overall solution would
go something like this: The effects runtime is implemented as a
stateless interpreter. The state of the virtual stack is maintained by generator scope. 
The transforms make no effort to thread that state
through the existing program. 

There are many reasons for making the interpreter
stateless. `effectsjs` started as a bolt-on dream, with the goal of having as
light of a touch as possible. Making the runtime globally-stateful is anathema
to that goal.

In my mind, the second we add global state behind the scenes is the
second the project becomes an interesting quirk, but nothing much more. Debugging mutable global state shared between
cooperatively scheduled tasks in the virtual stack seems _too hard._

In the absence of maintaining global state, what we actually need is a more
robust, heavier-handed compilation phase. Transforming the effects sugar needs
to thread the virtual stack through the program as it's utilized. The overall
solution almost definitely requires some code normalization and proper
compilation techniques as opposed to the more ad-hoc transpilation approach that
the project currently takes.

Most of the work will be taking place here, issues are inflight. A second area
for improvement is the actual grammar. The initial implementation borrowed from
Dan Abramov's pontifications from the original article cited above. I am unhappy
with this implementation overall and would much rather favor a functional,
expression-based syntax. The similarities of `throw/catch` semantics are
convenient for explaining the mechanics for uninitiated users, but the costs are
far too high.

### Interests at large, purpose of the new blog

Contrary to most of the contents thus far, it's not my intention to only focus on effectsjs in this blog. 

2020 has been a hard year for a lot of people. I wasn't an exception in this category, 
though I do feel fortunate to have come out on the other end relatively
unscathed. In the last week or so in doing some reflecting on where life has
taken me so far, and where I would like to see it go-- the one thing I regret is
not writing more.

I learned a great many things over the last year. Interests around PL research
have really gelled. Had I been writing about all of the info I soaked up along
the way, about the Category Theory, Algebraic Data Structures, realizations and discoveries made around
`effectsjs`, I'd be much farther along in my journey.

I'm not sure what kind of audience an obscure programming nerd-out blog has, but
there's only one way to find out I suppose. At the very minimum I'll understand the topics 
I'm interested in all of the better.

[^1]:[Algebraic Effects for the Rest of Us](https://overreacted.io/algebraic-effects-for-the-rest-of-us)
[^2]: [Continuations Coroutines Fibers Effects](https://medium.com/yld-blog/continuations-coroutines-fibers-effects-e163dda9dedc)
[^3]: [Algebraic Effects in Javascript](https://dev.to/yelouafi/algebraic-effects-in-javascript-part-1---continuations-and-control-transfer-3g88)
[^4]: [General Theory of Reactivity](https://github.com/kriskowal/gtor)
[^5]: [co](https://github.com/tj/co)
[^6]: [suspend](https://github.com/jmar777/suspend)
