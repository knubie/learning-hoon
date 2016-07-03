---
date: ~2016.6.23
author: Matthew Steedman
title: Learning Hoon - Part 4: Molds
type: post
navhome: /learning-hoon
navdpad: false
navmode: navbar
sort: 5
hide: true
next: true
---

In the previous post we explored gates and different ways of calling and making them. One thing we noticed is that making a gate requires things called `molds` which we don't know much about yet. So let's explore them.

## What's a mold?

According to the docs:

> ### mold (type, as constructor)
> A mold is a constructor function (gate). Its sample is any noun; its product is a structured noun. A mold is idempotent; (mold (mold x)) always equals (mold x).

Ok, so a `mold` is some kind of type constructor. It's a gate which means it's callable. Molds are used to define useful `spans`, but what's a span? Back to the docs:

> ### span (type, as range)
> A span is a Hoon type. It defines a set (finite or infinite) of nouns and ascribes some semantics to it. There is no syntax for spans; they are always defined by inference (ie, by mint), usually using a constructor (mold).

Ok, so what I've gathered so far is that `span` is a type, and `mold` is a type constructor. Admittedly I don't know what "range" means in this context. I'm sure there's a lot more to it, but that's all I understand at the moment.

If you remember from the previous post we used `@` as a "type" for an `atom` (an unsigned integer). The `@` is actually called an `aura` and there are a bunch of them. For example `@ux` is the aura for hexidecimal. Turns out you can also call them like gates.

```
dojo> (@ux 1)
0x1
```

I don't know if that means they are molds or what.

Let's go back to our friend `$=` which you may remember we used to "declare" the type signature for our gate. The docs describe `$=` as a "mold which wraps a face around another mold." According to the docs a `face` is "named variable."

The `$=` stem is part of a whole family of `$` runes that has their [own page in the docs](http://urbit.org/docs/hoon/twig/buc-mold/).

> A mold is a gate (function) that helps us build simple and rigorous data structures. (In fact, since "mold" sounds nasty, we often call molds and mold builders "structures.")

Ok we kind of already knew that. Let's play around with `$=` some.

```
dojo> ($=(a @ux) 1)
a=0x1
dojo> ($=(a @ux) 2)
a=0x2
dojo> (add 0x1 0x2)
3
```

So when we write `$=(a @p)`, it produces a `mold` (`gate`), which wraps a face, `a`, around another mold, `@ux` (the mold for unsigned hexadecimal). The thing is, `@ux` is also a mold, which means it's a `gate`, which means we can call it.

```
dojo> (@ux 1)
0x1
```

> Usually we use molds purely in the first sense: as an abstract definition of a noun. Don't actually call a mold unless you're actually validating untrusted foreign data. As a beginner, hopefully you aren't!


A lot of the documentation for different `stems` shows the bulb's legs as either `moss` or `seed`. What are they? Well to keep things brief, a `moss` is a twig that produces a `mold` and a `seed` is a twig that produces anything else. You can read more about `moss` and `seed` [here](http://urbit.org/docs/hoon/twig/).


---

those are functions whose range of possible outputs
are used to define a span at compile time
