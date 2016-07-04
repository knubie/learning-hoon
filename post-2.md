---
date: ~2016.6.23
title: Part 2: Cores
type: post
navhome: /learning-hoon
navdpad: false
navmode: navbar
sort: 3
next: true
---

In the previous post we explored gates a bit which are just "dry one-armed cores with sample". In Hoon, a `core` is a bit like an object in other languages. Its members/attributes are called `arms`. So now let's explore `cores` a bit.

## Let's make a gate (the long way)

When we created a `gate` earlier, what we really created was a `core` with one `arm`. Let's make a core now.

```
|%
++  bar
  "Hello!"
--
```

The first rune `|%` creates a new core, then `++` creates a new `arm` on that `core` named 'bar' with the value of "Hello!". Finally the `--` tells hoon we're done adding `arms`. Because `|%` is one of those rare stems that don't know how many children to expect, we need to end the twig with `--` when we're done. If we were to assign this `core` to a variable in the dojo like: `=foo |%  ++  bar  "hello"  --` we could then access its arm like this: `bar.foo`. Notice how the attribute accessor syntax is backwards compared to most other languages. If this `core` was written in javascript it might look something like this:

```
{ bar: "Hello!" }
```

So now that we know, roughly, what cores are, here's what our gate from [part 1](/post-1) looks like as a core.

```
:: This gate:
|=(a/@ a)
::
:: Is the same as this core:
=|  a/@
|%
++  $
  a
--
```

Let's go through this step by step. The first rune, `=|` essentially declares an uninitialized variable (`a/@`) to be used in the following twig (the [subject](/post-3), our `core`).

> ###### :new =| , "tisbar"
> `{$new p/moss q/seed}`: combine a defaulted mold with the subject.

In this case, `a/@` is our `p/moss`, and `q/seed` will be our core that we're creating. This allows the "defaulted mold" (`a/@`) to be available in the subject (our core). This variable will later act as the "argument" (or sample in hoonspeak).

Next, the `core` twig is similar to the one we made above, except instead of an armed name `foo` we're calling it `$`, and returning `a`.

And there we have it, a one-armed core with sample. Now we should be able to call this core just like we called the gate:

```
dojo> =foo =|  a/@  |%  ++  $  a  --
dojo> %-(foo 1)
1
```

It works! Notice I've compressed into a single line, but it's still considered "tall" form.

## Gates are a type of door

The truth is, gates are a type of core, yes, but more generally they are a type of `door`, (which is itself a type of core). A door is just like a gate, except instead of one, unnamed arm, it has multiple named arms. Here's what a door looks like:

```
=|  a/@
|%
++  add-one
  (add a 1)
++  add-five
  (add a 5)
++  add-ten
  (add a 10)
--
```

I should note that you can use `|_` to make doors explicitly in the same way we use `|=` to make gates explicitly without having to construct the core manually. Notice how `|_`, `|=` and `|%` all begin with `|`? That's because they all belong to the [Core family of runes](http://urbit.org/docs/hoon/twig/bar-core/).

```
|_  a/@
++  add-one  (add a 1)
++  add-five  (add a 5)
++  add-ten  (add a 10)
--
```

Notice that we've only declared one sample to be used amongst all the arms. Doors are unique in Hoon, in that there are no real equivalents in other languages. They're useful when you need to create multiple functions with the same signature. Assuming we store this core in a variable named `foo`, we can call its arms with with the rune `%~` (which is aptly named `:open`) like this:

```
dojo> %~(add-one foo 3)
4
dojo> %~(add-five foo 6)
11
```

Here's the documentation for `%~`:

> ###### :open %~ "censig"
> `{$cnsg p/wing q/seed r/seed}`: call with multi-armed door.

Here `p/wing` is the arm we want to call, `q/seed` is the core itself, and `r/seed` is the sample we're passing in.

Now, when we call a `gate` with `%-`, what we're really doing is using `%~` with the implicit parameter of `$` (the empty name).

```
dojo> =foo |=(a/@ a)
dojo> %~($ foo 1)
1
```

Notice the `$`? That's the single, unnamed arm of the core we made. In this case `foo` is our `gate` (remember, just a `core` with one `arm`), and `1` is the argument we're passing to it. The fact that the anonymous arm is actually called `$` means you can even call it again from within itself!

## A gate that opens itself

Let's see what a recursive gate looks like in hoon:

```
|=  {a/@ b/@}
?:  (lte a 10)
  $(a (add a 1))
a
```

There are a couple new things here. First the `?:` stem is essentially an if/else expression.

> ###### :if ?: "wutcol"
> `{$if p/seed q/seed r/seed}`: branch on a boolean test.

The first leg, `p`, is the condition to test, the second, `q`,  executes on true, and the third, `r`, executes on false. The other two new things are the `(add p q)` and `(lte p q) gates which are part of the standard library and should be pretty self explanatory (lte stands for less than or equal to).

The interesting part here is `$(a (add a 1))`, which calls the same empty-named arm (the gate itself, `$`) that it's inside of, essentially recursing. Curiously enough, though, the syntax is a bit different. We're not writing `($ (add a 1))` (the irregular form of `%-`). The syntax we're using is actually the irregular form of `%=`, which looks like `%=($ a (add a 1))` in regular form. Let's look at the documentation for `%=`.

> ###### :make %= "centis"
> `{$make p/wing q/(list (pair wing seed))}`: take a wing with changes.

Here, `p/wing` is referring to some "attribute", in this case `$`, an arm on a core. The following twig, `q/(list (pair wing seed))` is a list of `wing seed` pairs. We already know that a wing is like an attribute, and a seed could be anything. Those pairs represent the attribute we want to change, and what we want to change it to. Going back to our example, `%=($ a (add a 1))`, we have `$` as our `wing` and `a (add a 1)` as our `(list (pair wing seed))`. So within the context of `$`, we're changing `a` to be `(add a 1)`.

But why can't we just call `($ (add a 1))` like a normal gate? Because `$` *isn't* a gate. Remember, a gate is a *core* and `$` is an arm on that core (the only arm, in fact). Further, `%=` isn't just limited to arms on a core. It works for other `limbs` (attributes) as well. Take the following example:

```
=+  a=[b=1 c=2]  a(b 0)
:: returns [b=0 c=2]
```

Remember `=|`? It 'declares' an uninitialized (or, defaulted) variable to be used in the subject. As you may have guessed, `=+` is very similar but it declares an initialized variable. Now that that's out of the way, let's look at `a(b 0)` (the irregular form of `%=(a b 0)`. Here, `a` is our first wing. It's a wing (or attribute) of the entire expression (the subject), while `b` is a wing of `a`.

## Scope, how does that work?

Coming soon...
