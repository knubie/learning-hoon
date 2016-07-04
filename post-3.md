---
date: ~2016.6.23
title: Part 3: Subjects
type: post
navhome: /learning-hoon
navdpad: false
navmode: navbar
sort: 4
---

In part 2 we talked about cores, how they relate to gates, and how their attributes work. Part of that post dealt with declaring variables to be used within a core, as well as attributes (`arms`) of a core, and how they are related. In part 3 we'll explore the concept of variables, attributes, and scope within Hoon.

## Declaring a variable

In other languages, such as javascript, we can declare a variable  like this:

```
var a;
```

Once we've declared it, we're free to use it in the rest of the program, the *environment*:

```
var a;
{ foo: a }
```

In hoon, we can do something similar using `=|` to declare an "uninitialized" variable (actually it's a defaulted variable):

```
=|  a/@
|%  ++  foo  a --
```

But what happens when we try to use `=|` by itself?

```
dojo> =|  a/@
dojo<
```

Hmm, the dojo is expecting more stuff. Let's look at the docs for `=|`:

> ###### :new =| , "tisbar"
> `{$new p/moss q/seed}`: combine a defaulted mold with the subject.

As you can see, `=|` expects two children: `p/moss`, which is the variable to declare, and `q/seed`, the context, or *subject* to declare it in. 

> One feature Hoon lacks: a context that isn't a first-class value. Hoon has no concept of scope, environment, etc. A twig has one data source, the subject, a noun like any other.
>
> In most languages "variable" and "attribute" are different things. They are both symbols, but a variable is in "the environment" and a attribute is on "an object." In Hoon, "the environment" is "an object" -- the subject.

And if we want to declare two variables in a row? Since `|=` just modifies the subject and passes it along, we can use that modified subject as the subject for another `|=` twig. Here's a helpful graphic to illustrate the point:

<img src="https://dl.dropboxusercontent.com/s/k1qnge5s0e99j63/tisbar.png" alt="tisbar" width="600"/>

## Formulas

In Hoon, we call the code executed the "formula" and its context the "subject". Here is a more fleshed out example from the [arvo programming guide](http://urbit.org/docs/arvo/subject/):

```
::::::::::::::::::::::::::::::
=<  (sum [1.000 2.000])     :: formula
::::::::::::::::::::::::::::::
|%                          ::
++  three                   ::
  |=  a/@                   ::
  =|  b/@                   ::
  |-  ^-  @u                ::
  ?:  (lth a b)             ::
    0                       ::
  (add b $(b (add 3 b)))    ::
                            ::
++  five                    ::
  |=  a/@                   ::  subject
  =|  b/@                   ::
  |-  ^-  @                 ::
  ?:  (lte a b)             ::
    0                       ::
  ?:  =((mod b 3) 0)        ::
    $(b (add b 5))          ::
  (add b $(b (add b 5)))    ::
                            ::
++  sum                     ::
  |=  {a/@u b/@u}           ::
  (add (five a) (three b))  ::
--                          ::
::::::::::::::::::::::::::::::
```

In this example we're using `=<`, which is, obviously, different from `=|`, but the concept is similar (hence the common prefix `=`). In fact, all stems in the [flow family of twigs](http://urbit.org/docs/hoon/twig/tis-flow/) change the subject in some way.

> ###### :rap =< "tisgal"
> `{$rap p/seed q/seed}`: compose two twigs, inverted.

Essentially `=<` executes the first twig, (the formula, `p/seed`) in the *context* of the second twig (the subject, `q/seed`). That means whatever expression/twig we use as the formula will have access to all of the arms of the core we used as its subject. In the example above, the core gives us the context to call the `sum` gate as our formula.

Another thing you may have noticed in this example is that the `sum` arm in this core has access to the `five` and `three` arms. So how does that work with our formula/subject pattern?

> Each twig in the battery source is compiled to a formula, with the core itself as the subject. The battery is a tree of these formulas, or arms. An arm is a computed attribute against its core.

## An arm and a leg

Since cores contain their own context, we can access the attributes defined in its context. Remember when we used attribute accessor syntax `arm.core` to access an arm on a core? We can also use this syntax to access "variables" (named values, or values with a `face`) defined in its context:

```
dojo> =foo =|  a=5
dojo< |%  ++  bar  10  --

dojo> bar.foo
10
dojo> a.foo
5
```

Notice that we can access both `bar` and `a` the same way in the core `foo`. That's because they are both attributes (`limbs`) of the core, but there is a difference. `foo` is a "computed formula", so it's called an `arm`, while `a` is "defined as a subtree of the core", it's refered to as a `leg`.

Remember when we used the `(add a b)` gate that came from the standard library? You guessed it, the standard library is just a giant core that serves as the subject for the rest of our program, and `add` is just one of its many arms. In fact, here's the source code for `add`:

```
++  add
  ~/  %add
  |=  {a/@ b/@}
  ^-  @
  ?:  =(0 a)  b
  $(a (dec a), b +(b))
```

Can you figure out what's going on in this arm based on what we've learned in the previous chapters? (You can safely ignore the `~/  %add` line for now).
