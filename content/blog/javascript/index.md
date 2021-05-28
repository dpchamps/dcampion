---
title: A Sound Type System for Javascript
date: "2021-05-28T15:18:08.942Z"
description: ""
categories: [rambles, Pl Theory, Petal]
comments: true
draft: false
---

About a month ago, a coworker was updating our team after syncing with a third-party vendor. At one point they relayed something the vendor said, which was along
the lines of, "we opted to not use Typescript for [sensitive service] because we want a sound type system when writing sensitive code."

It makes sense, but my first reaction was, "that's odd." This is of course not a new idea. I write stuff in Rust all the time because of similar guarantees. However, so much critical software is written in Javascript that it feels odd to say out loud.

What makes Typescript and Flow (for example) different from the manifold compile-to-js alternatives is simply that they are supersets of ECMAScript -- by varying degrees. And that makes it much more palatable to businesses that have invested a ton of resources and time into web-based javascript applications.

Typescript's famous line that hooked everyone is that "all Javascript is valid Typescript." And I mean _c'mon_. That is catchy.

However, [A sound type system is a vocal non-goal of Typescript](), [Flow cares very deeply about soundness, but still strikes balances]().

Ryan Cavanaugh describes how he views the problem in this _very spicy_ thread:

> A fully-sound type system built on top of existing JS syntax is simply a fool's errand; it cannot be done in a way that produces a usable programming language. Even Flow doesn't do this (people will claim that it's sound; it isn't; they make trade-offs in this area as well). [^1]

He then goes on to say some stuff that I would categorize as _controversial_. Primarily, "JS runtime behavior is extremely hostile toward producing a usable sound type system."

How very interesting. But also probably not entirely wrong. Perhaps I shouldn't admit this, but I've been thinking about this 4 year old comment _a lot_ recently.

## Some Javascript is Valid [Type System]

Hypothesis: there is some intersection of Javascript and [Type System] that is both sound and delightful to work with.

_Note:_ This doesn't have the same value proposition as Typescript (or Flow).

Whatever this language is should not posture as a direct competitor to either. Instead, it should been seen as a supplement for the people who want to work with a pure type system, and/or for the use-cases that demand it.

_All javascript is valid Typescript_ is of course a bit of a gimmick. I love Typescript, and work with it professionally every day. In order to take the most advantage of the value-prop that TS offers, a user has to utilize a lot of the stuff that's _different_ from JS. Similarly,this type system would do the same. Skilled users would learn these features to become experts in writing efficient sound code that compiled to Javascript.

In a perfect world, the workflow would be:

- You write Javascript
- You follow clear compiler ( or language server ) errors until you have soundness.

So what would that intersection look like? **How much trimming would you have to do in order to start from something that wouldn't be what Ryan call's, "a fool's errand"?** On top of that, another question has to be kept in mind which is, **"what can we remove that isn't either loved or ubiquitous in the Javascript ecosystem?"**

## Candidates for removal

Here's a short list built off of nothing other than intuition, and my observations as a platform engineer:

1. `null`
1. `undefined`
1. `NaN` (?)
1. implicit coercion
1. `delete`
1. `for..in`
1. `let`
1. `var`
1. `getters/setters`
1. `class`
1. `throw`
1. `prototypical inheritance` (?)
1. `new` (?)
1. `instance of` (?)

`(?)` Denotes the union of, "should it" or "I want it to go away, but don't know if it can".

### `let` and not `const`.

This is a design decision based on the [idiom to prefer const](https://eslint.org/docs/rules/prefer-const). The idea is to provide something that's relatable to JS devs. The more comfortable it feels, the easier it is to adopt.

### `null` and `undefined`

If you ask JS devs what the difference between `null` and `undefined` are, they will probably parrot some adage, along the lines of "null is an assignment value, and undefined represents lack of assignment".

Both concepts I claim are meaningless to disambiguate, and are probably statically unenforceable, though I haven't given much thought to the latter part.

Anyway, the introduction of nully and unassigned values are part and parcel of the errand that said fool is embarking on -- so get rid of 'em.

### prototypical inheritence, `new`, `instance of` and `class`

Prototypical inheritance is a cool idea. It's unfortunately a big source of confusion in the language. Kyle Simpson of the _You Don't Know JS_ series makes a case for "Delegation-Oriented" Design, which would've probably been nicer than the popularity of classes, but nonetheless, it never caught on as an idiom.

Because of that, there was never really impetus to make that code syntactically more pallatable. At the end of the day, stuff like this is likely to make most JS Devs go "wait, wut?"

```javascript
const Task = {
    create(date){
        this.id = Math.random();
        this.dueDate = date;
    }
    print(){ console.log(this.id, this.date) }
}

const GetMilk = Object.create(Task);

GetMilk.create("tomorrow");
```

Intuitively, I feel that the people who recognize the benefits of writing code with a sound type system intersect with the FP-driven community of Javascript and are likely to feel at-home with these features removed.

For the rest, it should at least not be shocking to see. Given the current state of some of the more popular open source codebases and idioms in the community.

### What about the other stuff?

A detailed justification for each of these features I think justify separates posts.

Namely, exceptions and `NaN`. Everything else is either somehow related to another candidate for removal or so prone to side-effects or non-deterministic code that they're best left out.

---

All of this is just a _kernel_ of an idea. It's a rough sketch of mental notes, and a transposition of some actual notes that I've taken to organize think aloud about this central hypothesis. It's as far as I've gotten, which is admittedly not very far at all!

[^1]: https://github.com/microsoft/TypeScript/issues/9825#issuecomment-306272034
