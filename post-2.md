---
date: ~2016.6.23
title: Learning Hoon - Part 2: cores
type: post
navhome: /blog
---

In the previous post we explored gates a bit which are just "dry one-armed cores with sample". In Hoon, a `core` is a bit like an object in other languages. Its members/attributes are called `arms`. So now let's explore `cores` a bit.

When we created a `gate` earlier, what we really created was a `core` with one `arm`. Let's make a core now.

```
|%
++  bar
  "Hello!"
--
```

The first rune `|%` creates a new core, then `++` creates a new `arm` on that `core` named 'bar' with the value of "Hello!". Finally the `--` tells hoon we're done adding `arms`. If we were to assign this `core` to a variable in the dojo like: `=foo |%  ++  bar  "hello"  --` we could then access its arm like this: `bar.foo`. Notice how the attribute accessor syntax is backwards compared to most other languages. If this `core` was written in javascript it might look something like this:

```
{ bar: "Hello!" }
```

So now that we know, roughly, what cores are, here's what our gate from part 1 looks like as a core.

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

Let's go through this step by step. The first rune, `=|` essentially declares a variable to be used in the succeeding twig (our `core`).

> ###### :new =| , "tisbar"
> `{$new p/moss q/seed}`: combine a defaulted mold with the subject.

In this case, `a/@` is our `p/moss`, and `q/seed` will be our core that we're creating. This allows the "defaulted mold" (`a/@`) to be available in the subject (our core).

Next, `|%` creates a new core, `++` adds a new `arm` (attribute) onto the core called `$` (the empty name), and `a` is its subject of that arm (just returning `a` here without doing anything to it). Finally `--` is used to tell Hoon we're doing making our core.

And there we have it, a one-armed core with sample. Now we should be able to call this core just like we called the gate:

```
dojo> =foo =|  a/@  |%  ++  $  a  --
dojo> %-(foo 1)
1
```

It works! Notice I've compressed the core twig into a single line, but it's still considered "tall" form.

Now, when we call this `gate` with `%-`, what we're really doing is using `%~` with the implicit parameter of `$` (the empty name).

```
dojo> %~($ foo 1)
1
```

Notice the `$`? That's the single arm of the core we made. In this case `p` is our `gate` (remember, just a `core` with one `arm`), and q is the argument we're passing to it. The fact that the anonymous gate is actually called `$` means you can even call it again from within its own body!

```
|=  {a/@ b/@}
?:  (lte a 10)
  $(a (add a 1))
a
```

There are a couple new things here. First the `?:` stem is essentially an if/else expression.

> ###### :if ?: "wutcol"
> `{$if p/seed q/seed r/seed}`: branch on a boolean test.

The first leg, `p`, is the condition to test, the second, `q`,  executes on true, and the third, `r`, executes on false. The second new thing is the `(add p q)` gate which is part of the standard library and should be pretty self explanatory.

The interesting part here is `$(a (add a 1))`, which calls the same empty-named gate (`$`) that it's inside of, essentially recursing. (I'm not sure why `a` needs to be the first leg).
