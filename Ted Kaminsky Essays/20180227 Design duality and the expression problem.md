# Design duality and the expression problem

Feb 27, 2018

Part of what I hope to accomplish with this series is to make more concrete the nebulous notion of “trade-offs” that gets brought up any time program design comes up. So far I have brought up what I think are two of the most important ideas:

1. The most fundamental trade-off in abstraction design is [power vs properties](https://www.tedinski.com/2018/01/30/the-one-ring-problem-abstraction-and-power.html). And we humans seem to have a common bias towards more power, when getting the right properties is almost always more important. Adding a property limits power, though. Get comfortable with the idea! :)
2. The most fundamental piece of context is whether we’re [designing on a system boundary](https://www.tedinski.com/2018/02/06/system-boundaries.html). Most “rules” apply on a system boundary, where the code can’t be fixed later because it’d break external users. Outside of system boundaries, you can always fix mistakes later, so *“small, simple, clear, direct”* can outweigh adherence to rules.

I’ve also mentioned that part of my thesis is that our languages don’t do us any favors because [they routinely do not fully embrace both objects and data distinctly](https://www.tedinski.com/2018/01/23/data-objects-and-being-railroaded-into-misdesign.html). (If you want to bring up certain languages, you’ll probably find mention of them in that post, and stay tuned for future posts!) I’m going to make more than a few justifications for this over time, but today will be the first big one.

## A simple example

I’m going to pick something deliberately simple to start with, to make sure we all understand what’s going on. Here’s representing coordinates, where there’s two representations we want to support: polar and cartesian. We can make two different choices about how to represent this type: as an object or as data.

As an object, with slightly condensed Java-like notation:

```
interface Coord {
  float distance();
}
class Cartesian implements Coord {
  private float x, y;
  public float distance() { return sqrt(x*x + y*y); }
}
class Polar implements Coord {
  private float angle, dist;
  public float distance() { return dist; }
}
```

Or, as data, with Haskell-like notation:

```
data Coord = Cartesian { x :: Float, y :: Float }
           | Polar { angle :: Float, dist :: Float }

distance :: Coord -> Float
distance (Cartesian { x, y }) = sqrt(x*x + y*y)
distance (Polar { angle, dist }) = dist
```

Hopefully these are simple enough to understand without further explanation. And if you only understand one, you can transfer that knowledge to understand the other. There’s two important things to note right now:

1. In a closed world, these are identical representations.
2. The real world is rarely closed.

## The open world: extensibility, how?

There are two things that make our world open:

1. When other code imports our code and tries to use this type, what can they do?
2. When we want to modify our code in the future, in the face of outside users, what can we change without breaking them?

### The object world-view

With objects, we have a fixed interface. No new methods to it can be added without breaking third-party classes that implement that interface, because they will lack implementations. (However, new methods with default implementations can be added, for the somewhat rare cases when there is a sensible default implementation.) This means if we later decide we want to add an `angle` method, that’s going to be a breaking change for some users.

The upside is we’re free to add more representations. (Homogeneous coordinates, maybe?) Again, that’s both users of our library and ourselves in future iterations of the library. Further, we can make changes to the internals of our current representations, as long as they work the same way. (It’s hard to see the benefit of that with this example, but it can happen with more “real-world” examples.)

### The data world-view

With data, we have a fixed schema. There are just the two fixed representations we support for coordinates. Users cannot implement more of their own, and we cannot add more in the future without potentially breaking users, because they cannot handle the new case.

But what we can do freely is add more “methods.” That is, more functions over this type. Users are able to write `angle` on their own, and we’re able to add it to our library in the future without risking any problems.

### A third choice: the closed world-view, “abstract data types”

There’s another choice we can make, which is to actually close things off and not allow users to add new variants nor new operations. The common name for this is “abstract data type” which is actually a good name: we basically take the data implementation and make it “abstract” by hiding away the actual constructors of the type. Presumably, you’d add some functions that can construct values of that type, instead. (We can do ADTs with objects trivially, too. With our running `Coord` example, some languages allow “sealed interfaces” where the only implementations can be those in the same package. But otherwise just say “don’t implement this yourself, you’ve been warned.” Good enough.)

We’d choose to close down the extensibility of a type for several potential reasons:

1. We want to limit construction of the type. Consider `Date` and `ValidDate`. One is just data, the other acts like data, too, but also we also know the value won’t be “Feb 89th, -4294967296” because the only way to get one is through a function that verfies that first.
2. We want to limit the extent to which the data it wraps can be inspected. Consider taking a hash out of a database, and immediately wrapping it inside a `PasswordVerifier` abstract data type that never lets anything inspect the hash; it only exposes a `isCorrectPassword` method. (This doesn’t help with security, of course, that’s what the hashing is for. This helps ensure there’s exactly one place in the code that needs to change if your hash method ever changes. No other code could rely on what the hash looks like because they don’t get to see it.)

Extensibility-wise, we as authors gain the most from ADTs. Users cannot add new representations or operations, but *we* can do both in future iterations of our library, without breaking users. So we’ve limited users, but empowered ourselves. It’s a decent choice, plenty of the time.

*Notational quirk:* As a field, programming stinks at having good definitions of words. (See also: strong/weak typing.) Ah well, we’re a young field. I think there’s significant value to be had in clearly distinguishing between what I’m calling “abstract data types” and “objects.” But my use of these words is not universal. So let’s just note for a second:

- I call sealed (i.e. “no subclassing”) classes “abstract data types.”
- I call interfaces which new classes can freely implement “objects.”
- (And for that matter, I’m calling freely inspectable types “data.”)

Some sources ([like wikipedia](https://en.wikipedia.org/wiki/Abstract_data_type)) use “abstract data types” to refer to both what I call ADTs and objects, together. Other sources will also use “objects” to talk about both. My design aesthetic is closer to the latter: I think ADTs are best represented as classes not intended for subclassing. But we should think about them differently, so I’m trying to name them differently to help with that.

So I get a choice: argue about definitions of words, or invent new words. (“Closed objects”?) But I think wikipedia’s use of “ADT” is off. Few people would look at an interface and think “abstract data type” is a good name for that, right? And Liskov and Zilles’s paper [Programming with Abstract Data Types](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.136.3043&rep=rep1&type=pdf) appears to use terminology consistent with mine here.

But then I had to write this whole box about my notation, so maybe I’m just making things worse.

## The expression problem

![img](https://www.tedinski.com/assets/expression-problem.svg)

Two dimensions of extensibility for a type: variants and operations.

This choice about how a type can be extended is often called **[the expression problem](https://en.wikipedia.org/wiki/Expression_problem)**. It’s a slightly unfortunate name, to be honest. In the diagram above, we can see our two major design choices: objects and data. We also see two other things of note.

To get the first one out of the way, in the upper-left corner we see something strange. It’s not a box of its own.

Data leaks into the first box because even if we “close the world” to new operations, for many types it’s still possible to write arbitrary new functions anyway. Consider a `Pair` type with functions `fst` and `snd`. There’s nothing over `Pair` that you can’t write with those functions, so even if “closed” it’s still open. Types of this sort are just data. You happen to expose a “fold” or “universal function” or “recursion principle” for the type, and so anything can be done with it, just like data. (Indeed, if you think about how the visitor pattern works to emulate data with objects, this is exactly what’s happening.)

Likewise in the first box, objects leak in. Despite the name being “abstract *data* type” (and that name actually being a good description), the approach I advise is to use objects (sealed classes/interfaces if you have them) to represent ADTs. (Or at least, this is how I think we should do it in an ideal world where our languages didn’t get in our way. Use what’s appropriate for today.) An abstract data type is pretty much just an object you shouldn’t subclass. Objects already have the appropriate conventions: encapsulation of representation, constructors are functions we expect might throw, and so on. Often, we’d want to use both in conjunction in our designs: `Date` is data, `ValidDate` is an object that just wraps a date, but verifies it’s valid first. We can go back and forth between the two easily. But this makes it possible to pass around a date we know is valid, without having to resort to theorem proving or dependent types or something fancy.

The second thing to note, the part of this diagram that gives the expression problem its name, is the bottom-right corner. We programmers have rarely encountered something we couldn’t do, and *not* labeled it a problem to be solved. Why can’t we have extensibility in both ways? We want [more power!](https://www.tedinski.com/2018/01/30/the-one-ring-problem-abstraction-and-power.html)

### This design duality is fundamental

This is not a design problem that programmers have invented. It’s really a fundamental thing. An advanced alien civilization would likely have the same concept.

We see the same duality show up all over the place in mathematics. Sometimes you have data: the *natural numbers* are these things. Sometimes you have objects: *groups* are things that behave like this.

But one thing to note here is that mathematics as a field is not dominated by cults that limit themselves to one or the other representation. That part is unique to programmers. (Heh, watch someone prove me wrong. This isn’t to say they don’t find value in twisting their minds to think in certain ways. [Universal algebra](https://en.wikipedia.org/wiki/Universal_algebra) is going to be a lot more object-y, and [Number theory](https://en.wikipedia.org/wiki/Number_theory) is a lot more data-y. But as far as I can tell, this mostly changes what kinds of problems they try to solve, not how they develop their solutions.)

### Why don’t we want more power?

The expression problem has been “solved” in various ways. There certainly exist tools that give you extensibility on both axes. There’s a technique that works even in Java called “[Object Algebras](http://lambda-the-ultimate.org/node/4572).” There are several papers with “[a la carte](https://wadler.blogspot.com/2008/02/data-types-la-carte.html)” in the title. Or “[tagless final style](http://okmij.org/ftp/tagless-final/index.html).” Or, you know, just do whatever with dynamic languages.

Whatever else “well designed code” is, it’s definitely code that’s as easy to understand as possible. The trouble with the bottom-right quadrant of that design space, with extensibility over both variants and operations, is that some of these must be true:

1. Your code becomes extremely difficult to reason about. Or,
2. Your code becomes spaghetti, with ridiculous disorganization. Or,
3. There’s something very domain-specific you’re able to leverage to keep thing sensible.

The reasoning part isn’t too difficult to understand. With data, you can reason [inductively](https://en.wikipedia.org/wiki/Structural_induction), which, while it takes newbies some work to understand at first, it’s definitely something that qualifies as “pretty easy” once you get it. With objects and ADTs, you can think in terms of properties: preconditions, postconditions, invariants, and the like for simple cases. But the least terrible way to deal with extensibility in both direction is give up on modular reasoning entirely, think whole-program, and pick whichever of the above approaches you prefer (probably inductive reasoning). Going whole-program is a huge cost to reasonability, though. [Aspect-oriented programming](https://en.wikipedia.org/wiki/Aspect-oriented_programming) pretty much died under the strain of whole-program reasoning.

The spaghetti part is simple enough, too. If you need extensibility both ways, you already have: a base module, a module that adds a variant, and another module that adds an operation. And you probably also need the intersection: a module that adds the implementation of that new operation, on the new variant. Otherwise your program will crash if you use both. Enjoy filling in these voids and scattering little bits of disjointed implementation in different places.

*Story time:* My PhD thesis involved [a language](https://github.com/melt-umn/silver) with extensibility in both dimensions which probably qualifies for case 3. The domain was actually representing compiler abstract syntax trees, and we exploited some properties to make things work:

1. It’s compilers, so we could choose a data-heavy bias in the design. (Though I also include the dual representation as another option…)
2. It’s a language AST, so we could insist that new variants could proxy (or “forward” or “macro-expand”) to equivalent “host language” trees. Not always reasonable for just any data type.
3. I found modest restrictions that ensured we could always *safely* auto-generate the implementations at the intersection of new variants and new operations. So the spaghetti problem is mitigated, and as a bonus users can compose language extensions together and they “just work.” Nifty.

But another reason we don’t want that bottom-right quadrant is that we really often don’t need it. There’s more than a few situations where it seems like you might want it at first, but after some thought, you figure out you can break things down into a collection of types, variously represented as data and objects both, that can more accurately model what you’re after. That’s another interesting reason to want both objects and data in our languages, maybe I’ll examine this later.

## What can we do with our languages as they are?

With an object-oriented language, we can:

- Use objects for objects, hooray!
- Use objects for ADTs, no problem.
- Muddle with data in a few ways:
  - Sometimes we can use the [visitor pattern](https://en.wikipedia.org/wiki/Visitor_pattern).
  - Sometimes we can mangle things a bit to manage with an ADT instead. (See for example, the somewhat sub-optimal [Java `Optional`](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html).)
  - Sometimes we can get by with `enum`s together with other primitive data (arrays, records)
  - Sometimes we give up: our use case is small enough that it’s just too much boilerplate to bother with designing things as we “should” and we should pick a different design instead.

With a functional language, we can:

- Use data for data, hooray!
- For ADTs, just don’t export a data type’s constructors from its module.
- Muddle with objects in a few ways:
  - The simplest object is just a function closure after all. Sometimes that’s all we need.
  - Sometimes we can pass around records of closures (or function pointers) instead.
  - Sometimes we can change the design to use a single closure that responds to messages in the form of data. (In a sense, emulating having multiple methods with a single function.)
  - Sometimes we can get away with Haskell type classes or ML modules.
  - Sometimes we can pass the buck by being extra-parametric with our types (at the cost of bigger type signatures and sometimes more boilerplate.)
  - Sometimes we can do without by mapping into (or out of) wrapper types a lot.
  - Sometimes we give up, and try to make the design work with an ADT instead.

Eventually, I’ll write up more about all of these techniques. But today’s post is long enough.

## Summary

When we introduce a new type we have the choice of three fundamental designs:

1. Data. Expose the schema, extensible with more functions by users and in the future.
2. Objects. Expose interface, extensible with more implementations by users and in the future.
3. Abstract data types. Fully expose nothing, users cannot extend, but we can extend it in the future in both dimensions.

In an ideal world, we’d be encouraged to think in terms of these options (and to consciously choose among them) by our programming languages’ design. In the real world, we have to remember that no matter what language we’d chosen, none of these are best, and we sometimes have to work around the limitations imposed upon us.

([Continue reading in Part 2 of this post.](https://www.tedinski.com/2018/03/06/more-on-the-expression-problem.html))

#### Comments

[Comments section for Patreon backers](https://www.patreon.com/posts/17243752).

[Patreon](https://www.patreon.com/tedinski)

This post is part of a [book writing effort](https://www.tedinski.com/book/). Consider [supporting me on Patreon](https://www.patreon.com/tedinski).

#### Read more

- Follow me on Twitter: [@tedinski](https://www.twitter.com/tedinski/)
- Follow my [RSS Feed](https://www.tedinski.com/feed.xml)
- Check out the blog [archive](https://www.tedinski.com/archive/)
- Prev: [What would OOP without inheritance look like?](https://www.tedinski.com/2018/02/20/an-oo-language-without-inheritance.html)
- Next: [Notes on the expression problem and type design](https://www.tedinski.com/2018/03/06/more-on-the-expression-problem.html)

#### External Links

I noticed these discussions of this post: [Reddit](https://www.reddit.com/r/programming/comments/80ne6f/how_the_expression_problem_affects_program_design/)

Copyright © 2018 [Ted Kaminski](https://www.tedinski.com/about/).