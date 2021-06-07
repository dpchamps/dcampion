---
title: Using SWC For Extending JS
date: "2021-06-07T04:32:21.038Z"
description: ""
categories: [JS, ECMAScript, tools]
comments: true
draft: false
---

Last time, I talked about a toy language that could be an intersection of valid ECMAScript and a pure type-system.

Aside from doing some tinkering with what that type system might look like, I've been trying to imagine what _new_ types of constructs would exist in this language.

Pattern matching and algebraic data types were the first that came to mind. I got pretty excited thinking about how one might write the transforms for something similar to what rust `enum`s look like, and took a peak at the state of the [Stage 1 Pattern Matching Proposal](https://github.com/tc39/proposal-pattern-matching). It's alright.

For `effectsjs`, I wrote all of those transformations with a Babel Fork. The Babel internals are familiar enough to me that I could start iterating and experimenting quickly on these new transforms.

But I've been refraining doing anything with JS (or TS) because I see this project being a better candidate for Rust. For one, the Babel workflow is very slow. It's an older project, and I found the DX not very ergonomic. I'd like something _blazin_ fast, and to pass that speed down to the surely nonexistent customers using the language.

For another, I really like writing Rust and would like to write a bit more.

I started to embark on this path from the ground up. I wrote a somewhat functional Javascript lexer last summer, and figured I could build on it.

But I picked it up a few times, from a few different angles and kind of came to this conclusion that, "geez, this is a ton of work to get started." Kind of feeling like by the time that I'd have something satisfactory, all of the momentum of the initial project would be gone...

But then I had this thought, like _wait a minute, SWC exists!_

So I've been spending some time familiarizing myself with the SWC codebase. It's very impressive. At the moment it looks very promising for being able to do what I want. 

The downside is that I'm trading speed for complexity. Though I'm decent at Rust, I don't consider myself an expert -- and the SWC codebase feels _very Rusty_ (in a good way!).

However, after a weekend of wading around-- I think I've got a good enough grasp to take some baby steps. Seems like a natural progression would be to add a new token, get it to parse successfully, and then transpile it. Maybe even implement something like a curry operator. That seems easy enough. 

Will report my findings in the next post. Perhaps even do a cross post to medium-- who knows!