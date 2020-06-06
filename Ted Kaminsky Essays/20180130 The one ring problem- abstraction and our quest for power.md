# The one ring problem: abstraction and our quest for power

Jan 30, 2018

A quick story about me: I recently finished up a thesis related to extending programming languages with new features. One of the things I got to do along the way was take some time looking back on some old proceedings from conferences on that exact topic (Extensible Languages Symposium [1969](https://dl.acm.org/citation.cfm?id=1115858) and [1971](https://dl.acm.org/citation.cfm?id=942582)), to get familiar with what people have already thought up. While looking over them, I realized there was an unfortunate common theme:

Quite a lot of papers would come up with something they wanted to do, show that existing designs were incapable of doing it, then design some more powerful system where they could.

I believe this thought process is a common failing among programmers.

## The nature of abstraction

Abstractions are dualities, and in more ways than one. The classic example of the notion of a duality is light and darkness: two separate concepts, deeply inter-related, for the same underlying phenomenon. You can’t have one without the other. (Insert *Star Wars* fan theories here.)

People get flippant in saying that “everything is a trade-off” in programming, so much so that I think a lot of people start to see that statement as essentially meaningless. But abstraction design is the purest and most concrete expression of that cliché. You cannot create an abstraction without saying two things: what it is, and what it is not.

![img](https://www.tedinski.com/assets/abstraction-power-properties.svg)

The design spectrum: an all-powerful abstraction is a meaningless one (you’ve just got a new word for “thing”), while a tightly constrained abstraction could only be a few things. Design is figuring out how to find a point in the middle.

You cannot make an abstraction more powerful without sacrificing some properties that you used to know about it. Necessarily. You cannot require a new property be true about an abstraction without sacrificing some of its power and flexibility. Always.

The mistake is to forget this. Nearly always, this error happens in one direction. To look on a design, see what cannot be done with it, and attempt to “fix” it. To make it more powerful, and forget that it necessarily becomes more meaningless.

## “Help, I’m trapped in a function!”

Just to make sure we’re all on the same page, I’d like to show a small concrete example. Here are some simple functions, implemented in Haskell:

```
id :: a -> a
id(x) = x

inc :: Integer -> Integer
inc(x) = x + 1
```

These are quite trivial functions, but they’re already enough to be illustrative of something quite subtle. Abstractions have an inside and an outside. Both of these perspectives matter.

One easy mistake to make is to believe than an abstraction is just a common pattern: “Oh, I repeat this similar code a lot, so I’ll factor that out into a function.” But that’s exclusively looking at something like `inc` from an **outside** perspective. Inside of `inc`, there’s another abstraction: `x`. This variable isn’t a common pattern, it’s something else. It can vary, but there’s something we know about it. Even in a dynamic language, we’d have some notion that `inc` has to be used on something that makes sense to `+ 1`.

This inside/outside distinction is important, and it even shows up in a big way with `id`, as simple as that function looks. We want to be able to apply `id` to anything: `id(1) == 1` or `id("hi") == "hi"`. So we should be able to substitute anything in for those variables `a` we put in `id`’s type signature. But then consider an alternative implementation:

```
id :: a -> a
id(x) = 1
```

I hope you’re looking at that and thinking something is very wrong. What on earth would `id("hi")` do, return an integer we thought was a string? Well, perhaps obviously, this isn’t allowed: the type variables mean different things on the outside and the inside, and this implementation would be rejected with a type error. Outside, you can substitute them with anything, but inside they’re *held abstract*.

And being held abstract gets us properties: **there’s actually only one implementation of a function with the signature `a -> a`**, assuming no unsafe/magic, crashing, or infinite loops. (Haskellers call this neat trick “parametricity”).

We can also watch how power/properties shift together. We can’t presently add one to a floating point number, so what about *more power:*

```
inc :: Num a => a -> a
inc(x) = x + 1
```

Instead of being specific about integers, we allow any type, assuming that type is an instance of the `Num` typeclass (translation: “the type implements a `Num` interface that’s enough to know we can `+ 1`.”) But of course, now properties about `x` go away: we don’t know for a fact it’s an integer, so now this function probably has a less efficient implementation. On the other hand, we’ve also potentially gained some properties about `inc`: over integers, it could have done anything, but now it only knows about the operations that are part of `Num`. Trade-offs!

## Macros mean *more power*

Routinely, when faced with programmers who want power at all cost, language designers think “I know, I’ll add macros!” Now they have infinite problems.

Just to give a simple example, consider this small C program:

```
#define OH_NO(x) c_##x

int innocent(int c_x, int c_y) {
  return OH_NO(y);
}
```

Suppose you are tasked with developing a renaming refactoring for C. “How hard could it be?” After drowning yourself in the bottle merely over the problem of figuring out what prototypes in headers correspond to what implementations in C files, you come across the above example. “I want to rename `c_y` to `why_not`.” What do you do?

The correct answer is: quit your job, it’s not worth it, *life is worth living, save yourself!* Friends don’t let friends design languages with macro systems. We’ve made things more powerful, but we’ve lost really important properties that may well be much more useful. But more importantly, I think this decision is routinely made without understanding these consequences.

## We diagram plugin systems backwards

Everybody knows what a plugin system looks like:

![img](https://www.tedinski.com/assets/plugin-diagram-backwards.svg)

Obviously, the application with the plugin system is what’s important here. Plugins are just those little things you want to use along with it.

How does a plugin system work? Oh, that’s quite easy. You have your application’s mutable state, and you provide some event hooks where plugins can get called to go do *whatever* with it when an event happens. That’s eclipse, vim, emacs, and pretty much everybody.

What does that mean in terms of power and properties? Simple: plugins can do anything, and you know nothing at all about them. Which means our plugin diagram is backwards.

![img](https://www.tedinski.com/assets/plugin-diagram-corrected.svg)

The value of the plugin ecosystem sometimes overwhelms the value of the underlying application. As a result, the application becomes the small, constrained entity here: you can’t possibly make changes that would break plugins, users would become upset.

We see this everyday. Why is Python 2 still in use, when Python 3 is out? Well, the libraries (a similar phenomenon to plugins) are more important than the core of the system. Not all libraries work on the new system, so people stick to the old one.

Why is vim/emacs/eclipse still vim/emacs/eclipse? Even though, **? Well, same deal, handled differently: they can’t change the core of the system without breaking a plugin. It’s the application’s design that’s constrained, not the plugins. If emacs ever tries to truly fix its dynamic scope problem, I guarantee emacs will get forked.

## The one ring to rule them all

I could probably go on about programming languages: C++ and Scala both seem like dumping grounds for designs that are the product of “but then we can’t do *X*, so let’s make this even more powerful…” So let me pick a smaller kind of language: tiny DSLs or configuration file formats.

It’s a frequent occurrence that these start off as small declarative languages, and then as they evolve, they end up accidentally becoming Turing-complete. Generally, to their detriment. Sometimes, people advocate deliberately confronting this problem from the beginning: start off Turing-complete, just base your format on Lua or embed the DSL in Ruby or Haskell or something. That way, you’re not reinventing a general-purpose language, badly.

But… there’s still good reason to want your configuration format or DSL to be declarative. It goes from being data you can analyze to code that you have to execute. (Among a great many other things, imagine trying to deal with a file from an untrusted third party.)

One trick you can use to try to avoid this fate is to separate out *automation* from *declaration*. Encourage (and build tools to support) programmatic generation of the declarative file. Any time someone wishes they could write arbitrary code in the file, they may instead be able to write that arbitrary code to generate the file instead. This works surprisingly often, in my experience.

In the Bad Old Days, this was the best method to stay sane dealing with Jenkins: don’t manually create jobs, programmatically generate them. The job configuration could still stay declarative, but you can organize them however you need to in the code you write. (For example, to eliminate redundancy and not repeat yourself when you have a lot of similar jobs.)

These days, the Jenkins developers have done us one better: they figured out a good pattern and automated away job creation entirely. Now with multi-branch pipelines, the jobs are created for us, we can eliminate redundancies with shared libraries, and everything just works even better.

Lots of infrastructure works a bit like this. Kubernetes, for instance, has a declarative format for creating deployments/services/etc. It’s possible to write all these by hand, but it’s pretty much designed with programmatic generation in mind. This way, you can generate a configuration, diff it, commit it, push it to staging, and then push the exact same thing to the production cluster. When it’s exactly the same declarative data, you eliminate a lot of the ways that you could end up with a configuration that works differently in production than staging because it’s executing something differently. Instead of passing the execution around and hoping it gives the same results, you store the results, and pass them around directly.

Maven’s design is interesting, too, but that probably merits a new post.

## Resisting the temptation

The important thing here isn’t that these tools are good/bad. They’re just examples. The most important point is simply to recognize properties as being a valid (and arguably most important) part of abstraction design. We commonly, and I believe mistakenly, focus instead on power.

Quite a lot of designing programs well, in my experience, comes down to this: we have these common psychological quirks that lead us to bias our thinking in certain ways. To actually come up with good designs, we have to remember to actively work against our worst impulses. The scientific method isn’t some simple, clean, universal approach to finding the truth of our universe; it’s a rather messy, ad-hoc, *extremely human* approach to overcoming that flaws that cause us to believe things that aren’t true. To do design well, we have to work around ourselves, too.

This will also be a general theme of the book I’m trying to write.

#### Comments

[Comments section for Patreon backers](https://www.patreon.com/posts/16694713).

[Patreon](https://www.patreon.com/tedinski)

This post is part of a [book writing effort](https://www.tedinski.com/book/). Consider [supporting me on Patreon](https://www.patreon.com/tedinski).

#### Read more

- Follow me on Twitter: [@tedinski](https://www.twitter.com/tedinski/)
- Follow my [RSS Feed](https://www.tedinski.com/feed.xml)
- Check out the blog [archive](https://www.tedinski.com/archive/)
- Prev: [Data, objects, and how we're railroaded into poor design](https://www.tedinski.com/2018/01/23/data-objects-and-being-railroaded-into-misdesign.html)
- Next: [System boundaries: the focus of design](https://www.tedinski.com/2018/02/06/system-boundaries.html)

#### External Links

I noticed these discussions of this post: [Reddit](https://www.reddit.com/r/programming/comments/7u2bfo/the_one_ring_problem_abstraction_and_our_quest/)

Copyright © 2018 [Ted Kaminski](https://www.tedinski.com/about/).