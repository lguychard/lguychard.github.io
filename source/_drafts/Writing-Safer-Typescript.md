---
title: Writing Safer Typescript
author: Loïc Guychard
date: 2018-01-07 13:27:24
---



[Typescript](https://www.typescriptlang.org) is a wonderful little language. Note that, as Typecsript is just a superset of Javascript, some of the tips below are not necessarily Typescript-specific, and will be useful in writing safer vanilla Javascript as well.

## Good Typing is not Verbose Typing

A common misconception around Typescript, is that it is verbose: _"But doesn't your code end up litered with type annotation boilerplate?"_

Short answer: no, not really. Moreover, if it does, you are likely doing something wrong. Typescript's compiler is very good at inferring the type of your variables from their declaration. This means that, for exmaple

## `const` Everything

Mutable variables introduce a number of issues. [SICP]() provides a great summary: 

> programming with assignment forces us to carefully consider the relative orders of the assignments to make sure that each statement is using the correct version of the variables that have been changed. This issue simply does not arise in functional programs.

John Carmack (how's this for street cred?) concurs:

> I am a full const nazi nowadays, and I chide any programmer that doesn’t const every variable and parameter that can be. 

The outcome is easy: `const` everything you can inside your functions, and only use `let` when absolutely necessary. Code that makes heavy use of `let`, relying on encapsulation and local mutable state, is harder to reason about and more bug-prone.

As a rule of thumb, `let` should only be used to explicitly signal variables that, as an exception to the rule, can _and will_ be mutated in your block. This makes your code reviewers' job easier as well: on spotting a `let`-declared variable, he knows he will need to carefully review not only variable declaration, but also variable mutation.

## Never `any`, Always `never`

Enter `never`, `any`'s OCD righteous twin, who would've probably been right at home in a statically typed implementation of Vigil. From the [typescript handbook]():

`never`

## Use Precise Index Types

Javascript programs will often make extensive use of mapped objects, to store processing functions. See [this _Javascript Design Patterns_ entry](), this code here or that code there. Let's take the below example:

In older versions of typescript, the only possible types for index keys `string` or `number`.


## Enforce Purity

Javascript's most common data structures, `Array` and `Object`, are easily mutable by default:
 `Array.sort()` or `Array.splice()` mutate the array in-place, and unless one uses `freeze`, properties can be freely assigned or changed on an object.

This is coherent with Javascript's flexible accomodation of multiple paradigms, but is inconvenient when seeking to build a mostly pure codebase.

Typescript to the rescue! First, one may want to 

## Use `strictNullChecks` from The Get-go

By default in Typescript, `null` and `undefined` are valid values of all types. As a result, the following will compile without problems:

    let a: number = null
    let b: string = undefined

    const sayHi = (s: string) => `Hi ${s}!`

    sayHi(null) // will return 'Hi null!'

The problem is that 

This can lead to some subtle and evil bugs: I recently fixed a production bug that manifested itself with the following runtime crash:

    unedfined.then() is not a function

Oops! The 


## Treat Casting as a Code Smell

Casting a variable or value means instructing the compiler to treat it as a type it would not have initially inferred. In Typescript, this can be achieved with two different syntaxes. The first is the prefixed angled bracket annotation:

    saveRecord(<Record>{
        a: 'lol',
        b: 'michel'
    })

The second syntaxed is the postfixed `as`:

    saveRecord({
        a: 'lol',
        b: 'michel'
    } as Record)

Postfixed `as` casts can be chained, leading to the infamous `as any as T`: in the below example, the value passed to `saveRecord` would not validate the `Record` interface, as it is missing a `b` field. Thus, two vas

    saveRecord({
        a: 'lol'
    } as any as Record)

This, however, is not without its drawbacks.

## Conclusion

The easiest way to get started implementing these guidelines is to use the `safe-typescript-project` scaffold! This gives you:
- strict tslin.json and tsconfig.json
- pervasive non-mutable `Array` typings
- 
