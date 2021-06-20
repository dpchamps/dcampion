---
title: Doing Cool Stuff with TS Template Strings
date: "2021-06-19T22:53:51.693Z"
description: ""
categories: [Typescript]
comments: true
draft: false
---

Typescript 4.1 introduced a ton of really powerful features. [Template Literal types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html) is one that flew under the radar for me.When I first read about the release notes, I wrote TL Types off as interesting, but esoteric. The examples supplied cover what seem like frivolous edge cases to me. At the time, my impression was something along the lines of, "that's pretty cool I guess..."

But they're actually quite powerful. TL types allow for something novel: **static manipulation of string literal types.** Coupled with other new features in the type system, it enables the expression of types that were totally unavailable previously.

The way I stumbled upon this was by wondering if a type-safe `get` function (similar to something like the `lodash` library provides) could be typed with the new TS4 feature set. In this post, I'll build from abstract, to a concrete application. 

The abstract types are far more interesting and hold much more potential then the somewhat obsolete `get` function, which is just one example of where stuff like this might be useful. 

## Question: Can I Statically Join A Tuple of Strings into a Constant String Literal

Combining variadic tuple types, recursive types and template literals, we can get what we want:

```typescript
type Tuple<T extends unknown> = readonly T[];
type Tail<T extends Tuple<unknown>> = T extends readonly [any, ... infer U] ? U : [];

type Join<T extends Tuple<string>, Joiner extends string = "", Joined extends undefined|string = undefined> = 
  T[0] extends undefined ? Joined 
  : Join<
    Tail<T>, 
    Joiner,
    Joined extends undefined ? `${T[0]}` : `${Joined}${Joiner}${T[0]}`
  > 
```

It looks like a lot, so let's break it down line-by-line:

`type Tail<T extends Tuple<unknown>> = T extends readonly [any, ... infer U] ? U : [];`

This is a utility type that accepts an arbitrary tuple as a generic, pops the head off, and returns the rest.

Practically, it functions like so:

```typescript
type X = [1,2,3];
type Y = Tail<X>

// Y = [2, 3]
```

### The `Join` type

It's a [recursive conditional type](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html#recursive-conditional-types), and accepts one required generic and two optional generics:

`T extends Tuple<string>, Joiner extends string = "", Joined extends undefined|string = undefined`

* `T` must be a readonly tuple of string
* `Joiner` must be a string literal, and defaults to an empty string
* `Joined` must be either undefined or a string, and defaults to undefined

 We start by declaring a base-case:

`T[0] extends undefined ? Joined`

This is to say that if T is empty, then we return whatever we've collected.

Finally, we define the recursive step:

```typescript
: Join<
    Tail<T>, 
    Joiner,
    Joined extends undefined ? `${T[0]}` : `${Joined}${Joiner}${T[0]}`
  > 
```

Which says, recurse into the Join type. Pass the `Tail` of the tuple into the first paramater, pass through the `Joiner` and finally collect the first tuple element into the `Joined` string.

Let's see it in action:

```typescript
type X = ["1", "2", "3"];
type Y = Join<X, '-'>;

// Y = "1-2-3"
```

## If `Join` is Possible, Is the Inverse Also Possible?

If we can statically commute `Tuple` into `String` and preserve all of the literal data along the way, then it stands to reason that we'd be able to do to the inverse.

Up until this point, I'd be lying if I said that I was _super_ excited about these results. The magic here is all within the `infer`red variadic tuple in `Tail`:


```typescript
T extends readonly [any, ... infer U] ? U : []
```

Without this little bit of magic, all would be lost. But I've been familiar with this since the Variadic Tuple proposal, so nothing is very shocking here. It's still super cool. Just not new.

However, the big question is _can we deploy a similar technique to infer parts of template literal types?_

```typescript
type Test<T extends string> = T extends `abra${infer U}` ? U : "";

type A = Test<"abrakadabra">;
// "kadabra
type B = Test<"alakazam">
// ""
```

Now that is new to me!  With that experiment in mind, we can type something like:


```typescript
type Split<
  T extends string,
  Splitter extends string = "",
  U extends Tuple<string> = []
> = 
  T extends `${infer Head}${Splitter}${infer Tail}` ? Split<Tail, Splitter, [...U, Head]>
  : T extends "" ? U
  : T extends `${infer Rest}` ? [...U, Rest] 
  : never
```

### Breakdown

This is very similar to the `Join` type.

```T extends `${infer Head}${Splitter}${infer Tail}` ? Split<Tail, Splitter, [...U, Head]>```

Most of the magic lives in this line. This is sort of an in-line `Tail` function for our input: on the first substring to match, we grab the `Head`, and `Tail`, and recurse until we hit:

``` : T extends "" ? U ```, or
```T extends `${infer Rest}` ? [...U, Rest]```

Which is just to say, until we can no longer find the `SplitOn` substring. In which case, return the tuple we've been collecting if the rest of the string is empty, or the tuple plus the rest of the string.

`: never` indicates an exhaustive match to complement the third condition: ```T extends `${infer Rest}` ?```. A base case that we'll never encounter. 

Let's take it for a spin:

```typescript
type A = "a.b.c";

type B = Split<A, '.'>
// ['a', 'b', 'c']
type C = Split<A, "">
// ['a', '.', 'b', '.', 'c']  
type D = Split<"", "">
// []

type ident1 = Join<Split<"a.b.c", ".">, ".">
// "a.b.c"
type ident2 = Split<Join<['a', 'b', 'c'], '.'>, '.'>
// ['a', 'b', 'c']
```

## Sketching a type-safe `get` function

This was more of an exercise in _Can I_, rather than _Should I_. get-style functions are practically defunct now that optional chaining is so widely supported. Regardless, I still see requests from time to time to use it, and developers still seem to like it.

```typescript
type UnpackPath<T extends any, U extends Tuple> =
    U[0] extends undefined ? T 
    : UnpackPath<
        T[U[0]],
        Tail<U>
    >

declare function get<
  T extends unknown, 
  U extends string, 
  V extends unknown = undefined
>(o: T, props: U, def?: V) : UnpackPath<T, Split<U, '.'>> & V extends never ? UnpackPath<T, Split<U, '.'>> : V;
  
```

Let's start by breaking `UnpackPack` down:

We're using a similar technique to what's been covered so far: establish a base case and establish a recursive step. Compute the tail of a tuple, and winnow the initial tuple down to an empty case.

We take anything as the type's first parameter, and a tuple that will indicate properties to recursively lens into the first value.

```typescript
UnpackPath<
        T[U[0]],
        Tail<U>
    >
```

The recursive step takes the head of the tuple and recurses with a property access of the head, and passes along the tail.

`get` takes anything as it's first parameter, a string as it's second and optionally any value whatsoever as the third.

* `T` represents the input object
* `U` represents the property path that we want to retrieve
* `V` represents an optional value that the function will return if the property cannot be found.

```typescript UnpackPath<T, Split<U, '.'>> & V extends never ? UnpackPath<T, Split<U, '.'>> : V;```

The return type could perhaps use some work. This is why I called it a sketch. What we're saying here is that if the return type and default value cannot be unioned, than `UnpackPatch` must be returning a real value, otherwise the value must be unknown, which means the default value will be returned. There are some edgecases, especially around optional properties that I'm sure we could work around.

But here's what it looks like in action:

```typescript
type SomeRichType = {
  a: {
    deeply: {
      nested: {
        prop: string,
      }
    }
  },
  something: {
    with: {
      anArray: ['a', 'b', 'c','d']
    }
  }
};

const x: SomeRichType = {} as any;

// Access properties, shallow and deep
const a = get(x, "a");
    // { deeply: {...} }
const b = get(x, "a.deeply");
    // { nested: {...} }
const c = get(x, "a.deeply.nested.prop");
    // string

// Array access with numbers
const d = get(x, "something.with.anArray.1");
    // 'b'

// Typed Default Vals
const defaultVal = get(x, "this.dont.exist", 10);
    // 10
```