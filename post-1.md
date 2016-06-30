---
date: ~2016.6.23
title: Learning Hoon - Part 1: gates
type: post
navhome: /blog
---

I'm a programmer but I wouldn't say I'm a very good one. I don't have a computer science or math degree. I don't know much C or or low level systems programming. I just like to fiddle around with code and make things. So, join me as I fumble my way around learning Urbit's programming language: Hoon.

## Let's call a `gate`

According to the docs, a gate is like a function. The reason these aren't just called "functions" is to differentiate them from other mathematical-style functions in the Urbit world. For instance, a Nock formula is also a function (whatever that means!). So, here is one way to call a gate:

```
(add 2 2)
```

Looks a lot like lisp. Here's another way to do the same thing:

```
%+  add  2  2
```

Notice the double-spaces between nodes, those are called `gaps`. When we use `gaps` to delimit the nodes in our expression (also known as `twig` in hoon), we can omit the parentheses. So what's the `%+` symbol mean? It's called a `rune`, there are a lot of them in Hoon and they all do different things.

So what, specifically, does `%+` do? According to the docs it's called a `calt` or `centlus`, and it "calls with pair sample." That means is calls a `gate` with 2 arguments.

Going back to `(add 2 2)` and `%^  add  2  2` for a moment, what's the difference between these two syntaxes, why does one require a rune, and the other doesn't? The difference lies within a Hoon concept called "regular" and "irregular" forms (as well as "tall" vs "flat", which we'll explore later). `%^  add  2  2` is the *regular* form, while `(add 2 2)` is the *irregular* form. You can think of irregular form as essentially syntactic sugar. Here is some official documentation on "tall regular form."

> Tall regular form starts with the sigil, followed by a gap (any whitespace except ace), followed by a bulb whose twigs are separated by gap.

So let's unpack this for a moment.

> Tall regular form starts with a sigil

The `sigil` is just our `rune`: `%+`.

> followed by a gap (any whitespace except ace),

Again, a `gap` is just 2 or more spaces or a new line (an `ace` is one space).

> followed by a bulb whose twigs are separated by gap.

So a `twig` (our expression) is made up of two parts. The `stem` (our rune, `%+`), and the `bulb` (everything else). In this case our bulb is made up of the 3 other twigs, `add`, `2`, and `2`, which are also called `legs` of the `bulb`.

Now, that's *tall* regular form. So what about *flat* regular form? From the docs:

> Flat regular form starts with the sigil, followed by `pal` (`(`, left parenthesis), followed by a bulb whose subexpressions are separated by ace (one space), followed by `par` (`)`, right parenthesis).

Alright, so in flat regular form our `twig` looks like this:

```
%+(add 2 2)
```

Pretty self-explanatory. It's basically a lisp-style s-expression with a `sigil` prefix.


## Let's make a `gate`

Now that we've played around with different ways to call a `gate`, let's see if we can make one. According to the docs there's a `sigil` (`stem`) called `:gate`.

> ###### :gate |= "bartis"
> `{$gate p/moss q/seed}`: form a gate, a dry one-armed core with sample.

Don't worry right now what a "dry one-armed core" is, we'll get into that later. Take a look at this notation: `{$gate p/moss q/seed}`. That's our `twig`. The `$gate` is the `stem`, and `p/moss q/seed` is the bulb. `p/moss` and `q/seed` are *subexpressions*, or `legs` of the bulb. We'll explain what `moss` and `seed` mean later. For now, let's use `|=` to make a simple gate.

```
|=(a/@ a)
```

Alright, there's our gate. You may have noticed I'm using the *flat* regular form here. The first twig, `a/@` is our `sample` or argument. It's name is `a` and `@` is its type. The `@` type (called an `atom`) is basically just an unsigned integer of any size. The second twig is the function body. Here I'm just returning `a` without doing anything with it.

So now that we have a gate, let's try and call it. We used `%+` (cenlus) before to call `add`. But if you recall `%+` is only used to call a gate with a pair sample. Our new gate we just made only has one sample, so we'll use `%-` (cenhep) instead (notice how they both begin with `%`, this is not by accident. `%^` calls a gate with triple sample). Let's briefly look at the documentation for `%-`.

> ###### :call %- "cenhep"
> `{$call p/seed q/seed}`: call a gate (function).

Again we have two samples (both seeds). The first, `p` is the `gate` we wish to call, and `q` is the argument. Let's try it.

```
%-(|=(a/@ a) 5)
```

Again, I'm using the flat, regular form for `%-` here. The first twig is the `gate` we created before, and the second twig is the sample `5`. If you recall from the previous seciton where we used `%+` to call `add`, the first twig was `add` which is gate that's build into the Hoon standard library. So, essentially here we are forming a `gate` and calling it in place. Neat!

Now let's modify our example to take two `samples`.

```
%-(|=(a/@ b/@ %+(add a b)) 1 2)
```

Oh, that didn't work.. Why not? Let's take a look at the `|=` twig by itself, this time in tall-form to make it clear:

```
|=  a/@  b/@  %+(add a b)
```

Remember, `|=`'s bulb only has two twigs, but here we're giving it a bulb with *three* twigs: `a/@`, `b/@`, and `(add a b)`. Right, so to make this work we need to combine our arguments `a/@` and `b/@` into a single twig. Remember the definition of `|=`?

> `{$gate p/moss q/seed}`

When we combine our first two twigs into one, it needs to be a `moss`. What's `moss`?

> We use moss to describe twigs whose product is used as a mold, and seed for twigs whose product could be anything.

What's a `mold`? It's essentially a type constructor. `a/@` is *moldy* (it's a `moss`) because its product is a mold. Without worrying too much about what a mold is, we just know we need a moldy twig to replace these two `a/@` and `b/@` parameters. The `%:` stem let's us pass in a list of `moss` that creates a mold which will recognize a tuple argument.

> ###### :bank $: "buccol"
> `{$bank p/(list moss)}`: form a mold which recognizes a tuple.

Here we see that its only sample is `p`, which takes a list of `moss`, in our case `a/@ b/@`. And we know it works as `moss` because it "forms a mold," so let's try it.

```
%-(|=($:(a/@ b/@) %+(add a b)) 1 2)
```

Hmm, still doesn't work. Oh, right. We're still using `%-` to call the gate, which expects a single sample. Let's try `%+` instead.

```
%+(|=($:(a/@ b/@) %+(add a b)) 1 2)
```

Great, it works! I wonder if we can still use `%-`. Let's try combining the two samples into a tuple, since our gate has a mold that recognizes tuples.  According to the docs, `:-` constructs a cell (2-tuple).

> ###### :cons :- "colhep"
> `{$cons p/seed q/seed}`: construct a cell (2-tuple).

Let's try it:

```
%-(|=($:(a/@ b/@) %+(add a b)) :-(1 2))
```

Ok, that worked. I wonder if we can use this method on `add` as well?

```
%-(add :-(1 2))
```

Sure can! Let's back up for a moment. You may have noticed something in the previous code samples that looks.. out of place. It's these: `a/@`. They don't fit the regular syntax convention of `:sigil(bulb)`. That's because they're not regular, they're actually the irregular form of `$=` (buctis, or `:coat`). Rember irregular form is just syntactic sugar. The regular flat syntax for `a/@` is actually `$=(a @)`. Here's the documentation for `$=`:

> ###### :coat $= "buctis"
> `{$coat p/@tas q/moss}`: mold which wraps a face around another mold.


Again we see that `$=` is moldy just like `$:` (and they both start with `$`, see pattern?

And so the totally regular (and flat) version of our last code sample is:

```
%-(|=($:($=(a @) $=(b @)) %+(add a b)) :-(1 2))
```

That's quite verbose! Let's see what else we can convert to irregular form. First we'll convert our `$=` twigs back.

```
%-(|=($:(a/@ b/@) %+(add a b)) :-(1 2))
```

According to the docs, the irregular form of `$:(a b)` is `{a b}`.

```
%-(|=({a/@ b/@} %+(add a b)) :-(1 2))
```

That's starting to look better. If you recall from our previous experiments with `add` we can use the irregular form of `%+` and `%-`:

```
(|=({a/@ b/@} (add a b)) :-(1 2))
```

Turns out `:-(1 2)` also has an irregular form: `[1 2]`.

```
(|=({a/@ b/@} (add a b)) [1 2])
```

But since we're using the irregular form of `%-` we don't even need to use it at all!

```
(|=({a/@ b/@} (add a b)) 1 2)
```

Next try this: convert this into tall regular form, or a mixture of tall, flat, regular and irregular.
