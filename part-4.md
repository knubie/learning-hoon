---
date: ~2016.6.23
title: Part 4 - Molds
type: post
navhome: /learning-hoon
navdpad: false
navmode: navbar
sort: 5
---

In [Part 1](/part-1) we used `foo/@` to declare a sample for our gate named `foo` of span (or, type) `@`. So what exactly is `@`? We used it as sort of type annotation in our sample, but it's called a `mold`, or "type constructor". And because it's a type constructor, that means it's also a gate, which means we can call it:

```
dojo> (@ 'foo')
7.303.014

dojo> (@ ~halsed)
60.875
```

Wait, what are we doing here? What is 'foo'? What is `@`? Why does it produce `7.303.014`? Let's back up for a moment.

## Nouns

In Hoon, data values are called `nouns`. So what are they? They're either an `atom` (an [unsigned integer](https://en.wikipedia.org/wiki/Signedness)), or a `cell` (a pair of two `nouns`). Here are some nouns:

```
42        :: an atom, but also a type of noun
[42 42]   :: a cell (another type of noun) with two atoms (that are also nouns)
[1 [2 3]] :: a cell with an atom, and another cell (that has two more atoms)
'foo'     :: another atom
```

Wait, that last one doesn't look like an integer. It looks like some kind of string. In Hoon it's called a `cord`, an ASCII string, and as you may or may not know, ASCII is the numerical representation of characters like 'a', and '%'. This is useful because Nock, (and computers in general) only understand numbers. Remember that symbol, `@`? That's the symbol, or `aura` for an `atom`. That's why `(@ 'foo')` spit out an integer. This conversion works because `'foo'` is, in fact, also an atom! But it's a specific kind of atom called `@t` (another `aura`), AKA "cord".

We can view the `aura` of an atom in the dojo using `?`.

```
dojo> ? 'foo'
  @t
'foo'

dojo> ? 98
  @ud
98

dojo> ? 0xdead
  @ux
0xdead

dojo> ? 0xd0f8
  @ux
0xd0f8

dojo> ? -10
  @sd
-10
```

As you can see there are a number of these auras. Here are all of the auras that come with Hoon:

```
@                atom
 @c              UTF-32 codepoint
 @d              date
   @da           absolute date
   @dr           relative date (ie, timespan)
 @f              yes or no (inverse boolean)
 @n              nil
 @p              phonemic base (plot)
 @r              IEEE floating-point
   @rd           double precision  (64 bits)
   @rh           half precision (16 bits)
   @rq           quad precision (128 bits)
   @rs           single precision (32 bits)
 @s              signed integer, sign bit low
   @sb           signed binary
   @sd           signed decimal
   @sv           signed base32
   @sw           signed base64
   @sx           signed hexadecimal
 @t              UTF-8 text (cord)
   @ta           ASCII text (span)
     @tas        ASCII symbol (term)
 @u              unsigned integer
   @ub           unsigned binary
   @ud           unsigned decimal
   @uv           unsigned base32
   @uw           unsigned base64
   @ux           unsigned hexadecimal
```

As you can see they are all subsets of `@`, and in fact some of them are subsets of each other. For instance, `@tas` is a type of `@ta`, which itself is a type of `@t`, which finally is a type of `@`. That means any `gate` which expects a `@t` sample, will also galdly accept a `@ta` or `@tas` one.

> Auras are a lightweight, advisory representation of the units, semantics, and/or syntax of an atom. An aura is an atomic string; two auras are compatible if one is a prefix of the other.


