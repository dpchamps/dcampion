---
title: A Sound Type System for Javascript
date: "2021-05-28T15:18:08.942Z"
description: ""
categories: [rambles, Pl Theory, Petal]
comments: true
draft: false
---

About a month ago, a coworker was updating our team after syncing with a third-party vendor. Something they said stood out to me:

> We opted to not use Typescript for [sensitive service] because we want a sound type system when writing sensitive code.

It makes sense. But my first reaction was, "that's kinda odd." This is of course not a new idea. I write stuff in Rust all the time because of similar guarantees. However, so much critical software is written in Javascript that it just kind of feels odd to say out loud. 

Probably the most important feature of the Typescript and Flow type-systems is that they are very close to JavaScript. Typescript calls itself a superset, and Flow calls itself a static type-checker. This feature separates them in a big way from the manifold compile-to-js alternatives by being much more palatable to users that have invested a ton of resources and time into web-based JavaScript applications.

Typescript's famous line that hooked everyone is that "all Javascript is valid Typescript." And I mean _c'mon_. That is catchy.

However, [A sound type system is a vocal non-goal of Typescript](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Design-Goals#non-goals), [Flow cares very deeply about soundness, but still makes tradeoffs](https://flow.org/en/docs/lang/types-and-expressions/#toc-soundness-and-completeness).

[Ryan Cavanaugh (TS Development Lead)](https://github.com/RyanCavanaugh) describes how he views the problem in this _very spicy_ thread:

> A fully-sound type system built on top of existing JS syntax is simply a fool's errand; it cannot be done in a way that produces a usable programming language. Even Flow doesn't do this (people will claim that it's sound; it isn't; they make trade-offs in this area as well). [^1]

He then goes on to say some stuff that I would categorize as _even more controversial_. Primarily, "JS runtime behavior is extremely hostile toward producing a usable sound type system."

How very interesting. But also probably not entirely wrong. 

Somewhat embarrassingly, I've been thinking about this 4 year old comment _a lot_ recently.

## Some Javascript is Valid [Type System]

Hypothesis: there is some intersection of Javascript and [Type System] that is both sound and delightful to work with.

_Note:_ This doesn't have the same value proposition as Typescript (or Flow).

Whatever this language is, it should not posture as a direct competitor to either. Instead, it should been seen as a supplement for the people who want to work with a pure type system, and/or for the use-cases that demand it.

_All javascript is valid Typescript_ is of course a bit of a gimmick. I love Typescript, and work with it professionally every day. In order to take the most advantage of the value-prop that TS offers, a user has to utilize a lot of the stuff that's _different_ from JS. Similarly,this type system would do the same. Skilled users would learn these features to become experts in writing efficient sound code that compiled to Javascript.

In a perfect world, the workflow would be:

- You write Javascript
- You follow clear compiler ( or language server ) errors until you have soundness.

So what would that intersection look like? **How much trimming would you have to do in order to start from something that wouldn't be "a fool's errand"?** On top of that, another very important question is, **"what can we remove that isn't either loved or ubiquitous in the JavaScript ecosystem?"**

## Ideas

Here's a list of things we might cut out, built off of nothing more than intuition, and my observations as a platform engineer for a very website.

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

`(?)` Denoting the union of, "should it" and "I want it to go away, but don't know if it can".

### `let` and not `const`.

This is a design decision based on the [idiom to prefer const](https://eslint.org/docs/rules/prefer-const). The idea is to provide something that's relatable to JS devs. The more comfortable it feels, the easier it is to adopt.

### `null` and `undefined`

If you ask JS devs what the difference between `null` and `undefined` are, they'll likely same something along the lines of, "null is an assignment value, and undefined represents lack of assignment."

I think I would claim that the two concepts meaningless to disambiguate, and are statically unenforceable, though I haven't given much thought to the latter part.

We can do without nully and undefined values, which are part and parcel of the errand that said fool is embarking on -- so get rid of 'em.

### prototypical inheritance, `new`, `instance of` and `class`

Prototypical inheritance is a cool idea. It's unfortunately a big source of confusion in the language. Kyle Simpson of the _You Don't Know JS_ series makes a case for "Delegation-Oriented" Design in "this & Object Prototypes." This pattern probably would've been nicer than the pattern of classes that got popular, but nonetheless, it never caught on as an idiom.

Because of that, there was never really impetus to make that code syntactically more palatable. At the end of the day, stuff like this is likely to make most JS Devs go "wait, wut?"

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
GetMilk.print();
```

Intuitively, I feel that the people who recognize the benefits of writing code with a sound type system intersect with the FP community within Javascript and are likely to feel at-home with these features removed.

For the rest, it should at least not be shocking to see. Patterns for encapsulation of data are as old as [Addy Osmani's JavaScript Design Patterns](https://addyosmani.com/resources/essentialjsdesignpatterns/book/), and [Douglas Crockford's JavaScript: The Good Parts](https://www.oreilly.com/library/view/javascript-the-good/9780596517748/). 

### What about the other stuff?

A detailed justification for each of these features (including those that I outlined above) deserve separate posts.

Anyway, all of this is just a _kernel_ of an idea. It's a rough sketch of mental notes, and some actual notes that I've taken to organize thoughts about this central hypothesis. 

It's as far as I've gotten, which is admittedly not very far at all!

[^1]: https://github.com/microsoft/TypeScript/issues/9825#issuecomment-306272034
