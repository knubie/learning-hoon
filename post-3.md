---
date: ~2016.6.23
author: Matthew Steedman
title: Learning Hoon - Part 3: Subject
type: post
navhome: /learning-hoon
navdpad: false
navmode: navbar
sort: 4
next: true
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

We can actually prove that "variables" in hoon are attributes by using the attribute accessor syntax we saw earlier:

```
dojo> =bizz [c=1 d=2]
dojo> c.bizz
1
dojo> =foo =|  a/@  |%  ++  bar  a --
dojo> bar.foo
0
dojo> a.foo
0
```

And if we want to declare two variables in a row? The context created by the second variable declaration becomes the subject for the first, as illustrated here:

<img src="https://dl.dropboxusercontent.com/s/k1qnge5s0e99j63/tisbar.png" alt="tisbar" width="600"/>

In Hoon, we call the code executed the "formula" and its context the "subject". here is a more fleshed out example from the [arvo  tutorial](http://urbit.org/docs/arvo/subject/):

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

In this example we're using `=<`, which is, obviously, different from `=|`, but the concept is similar (hence the common prefix `=`).

> ###### :rap =< "tisgal"
> `{$rap p/seed q/seed}`: compose two twigs, inverted.

Essentially `=<` executes the first twig, (the formula, `p/seed`) in the *context* of the second twig (the subject, `q/seed`).
