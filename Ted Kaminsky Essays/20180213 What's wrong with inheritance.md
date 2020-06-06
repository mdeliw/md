# What's wrong with inheritance?

Feb 13, 2018

Over fifty years ago, we saw the first `class`. This was [Simula 67](https://en.wikipedia.org/wiki/Simula).

At the time, programming was quite primitive. The concept of defining new types was brand-new in the form of records (i.e. structs). Previously, essentially all types were built into the language and compiler. To modern eyes, a lot of the language used, and the problems and priorities people had, seem strange. They were still in the process of figuring it all out, after all. (Simula 67 was once introduced as “an extensible language” where you could “extend the language with new types” by using this `class` keyword… which is not how anyone would describe classes today.)

During this development, a neat type punning trick was embraced. If you have a record laid out in memory, you can happily pretend it consists of only its prefix. That is:

```
struct base {
  int a;
}
struct derived {
  struct base header;
  int b;
}
```

With such a setup, one can happily cast a `struct derived *` to a `struct base *` and everything is okay. This *cute trick* was embraced and leveraged into a language feature called **inheritance**.

## What went wrong?

This feature enabled a legitimately new and useful idea: dynamic dispatch. Seeing that dynamic dispatch was useful, and inheritance was the tool that gave it to us, legions embraced the design and went to town. Since inheritance brought us dynamic dispatch, inheritance must be good right? Decades have been spent trying to reconcile these ideas.

Generations of programmers have been raised with inheritance as one of the central features of their programming languages. They’ve been taught inheritance is about re-use, despite all the practical experience to the contrary. Design philosophies have embraced “is a” relationships in writing our code. Better design philosophies have turned around and detailed all the ways this kind of thinking also trips us up and encourages mistakes, but nevertheless seem shy about dismissing it entirely.

Quite quickly after the arrival of inheritance, it became clear it was too limited. Sometimes we wanted classes to be two different things. Our type punning trick only provides for one! And so we invented **multiple inheritance**.

This came with a multitude of problems. I’m not even sure I need to talk about them. If you haven’t used multiple inheritance, you’re still probably aware there’s a reason for that. If you have used multiple inheritance, you already know how bad it is. If you want to discover just how deep the rabbit hole goes, I suggest looking deeply into how C++ does it.

Faced with the same problem of wanting to “be” multiple things, and the failure of multiple inheritance (really, this should have been a clue), we sought out other ways to cope. This gave rise to the invention of **interfaces**. Interfaces avoid all the problems of multiple inheritance through a simple restriction: no data members, and equally no implementations of methods that use data members. As a result, all interfaces do is say “this is a thing that can dynamically dispatch on these methods.” (Incidentally, here is another design that is [vastly improved by limiting its power](https://www.tedinski.com/2018/01/30/the-one-ring-problem-abstraction-and-power.html).)

With this, we have essentially the modern formulation of object-orientation. Exemplified by Java, you have a class and it can implement any number of interfaces. But we still allow classes to single-inherit from a base class.

## What’s actually wrong with inheritance?

### Coupling interface and implementation

A class that is design to be inherited from is doing two things: it defines an interface through which any of its subclasses can be used, and it defines an implementation. But it also bundles these two things together into one class. This has a few negative effects:

1. Any implementation of that interface is forced to also accept the base class’s implementation.
2. The base class’s implementation frequently becomes fragile and difficult to change (more on that in a bit).
3. We can no longer have several alternative base implementations on equal footing.

Modern OO design methodologies have recommendations for working around this problem. Interfaces can be defined separately, and a base class implementation of that interface provided along side. This has a cost, in current languages, of making things a bit more verbose, but it does resolve the problems above. Unfortunately, this also comes with additional human-factor problems. Rote adherence to rules can lead to names like `IThing` and `ThingBase` or application in contexts when perhaps this separation was totally unnecessary.

### Fragile base classes and open recursion

The fragile base class problem should be relatively easy to understand. Base classes have reasonably precise specifications of their public interfaces (and “protected” interface, if your language supports such a thing). This is a good thing—it is the intermediary between telling classes what they have to provide, and telling users what they can make use of. Everyone gets along fine.

But there is no precise specification of how the base *implementations* of methods make use of the other methods.

We can start with the following Java/C-ish pseudo code:

```
class Writer {
  void putch(char ch) { ... }
  void putstr(char *str) { ... }
}
class SimpleWriter extends Writer {
  void putch(char ch) { char buf[2] = { ch, 0 }; putstr(buf); }
}
```

And later we can look at `Writer` on its own and think “hey we could make this implementation simpler”

```
class Writer {
  void putch(char ch) { ... }
  void putstr(char *str) { while(*str++) putch(*str); }
}
```

This change to the base class creates infinite recursion and breaks `SimpleWriter`. But there’s no way to know we’ve done anything wrong until we look at or test `SimpleWriter`. But subclasses may not be under our control or visible to us, they may be in user code who knows where. That’s part of the whole point of objects—they’re open to extension—isn’t it?

Without such a specification of how methods use each other, there’s no agreed upon understanding of what subclasses should expect in terms of behavior from their super class. Thus, subclasses could come to expect almost anything true of the current base class implementation, and almost any change in behavior in the base class could then break the behavior of a subclass. Thus, fragile base classes.

The reason separating interface from implementation mitigates this problem isn’t that it truly solves anything. Instead, we can employ the (possibly rather ugly) solution of just providing another alternative base class and leaving the old one alone. You can then hope that subclasses migrate over to the new and improved base implementation (hopefully not named `ThingBase2Fixed_Use_This_One_V3`) while leaving the old one alone, confident we haven’t broken anything.

This problem gets blamed on **open recursion**. Open recursion is the technical name for how late-binding gets done in classes. When the base class calls another method on itself, expecting to get its own implementation, and gets a subclass implementation instead, that’s the general idea of what open recursion enables.

There are a lot of people who believe the open recursion is actually an all-around terrible idea. I’m not entirely sure I’m on board with that claim yet, but I do have to admit it can cause problems. The open question, to me, is whether the cure is worse than the disease. Open recursion certainly plays hell with formal verification efforts, but in practice, nearly every design that relies on it uses some special case with a reasonable workaround. I have a vague suspicion there’s probably some restricted form of mixins that’s actually a pretty nice replacement.

### Variance, and the uselessness of “is a”

Variance is an inherent aspect of subtyping. Suppose we have a small subtyping relationship: `num :> int :> nat`. So these types are going from more general to more specific: any number, just integers, just natural numbers. What types are acceptable subtypes of the function `int -> int`?

If you think about it for a bit, you realize you can be less precise about the output of the function. So `int -> num` is okay, the function doesn’t need changing or anything. Likewise, we can artificially be more restrictive of the inputs and the function can still handle it fine. So `nat -> int` is okay, and putting them together `nat -> num` is okay.

But not the other way around. If you tried to cast the function to `num -> int` what would it do when called on non-integers? It wasn’t written to understand them. *Crash*.

This interplays with generics as well. Consider `Source` that we can `get` a value from. Clearly if we have a `Source` we can treat that like a `Source` without issue, but not a `Source` since that may now give us unexpected values. Similarly, if we have `Sink` that we can `put` things into, the behavior is the opposite. We could have a `Sink` but now `Sink` is the problematic one because you could now put unexpected values into it. One is *covariant* and one is *contravariant*.

I chose `Source` and `Sink` for a reason: state makes types *invariant*. Assigning to a mutable reference is like `put`, while accessing a mutable reference is like `get`. This means something mutable may not really have subtypes at all.

This relatively simple concept scales up, and hits us hard. Too many OO books start with some uselessly dumb example of inheritance (“car extends vehicle” is the punch line of a joke at this point). Nearly all of these are heinously terrible designs.

Let’s look at a [classic example of naive “is a” relationships: shapes](https://en.wikipedia.org/wiki/Circle-ellipse_problem). Squares are special case of rectangles, right? But making `Square` a subtype of `Rectangle` puts two methods for mutating length and height on our squares, which is a bit weird. We can try violating the “is a” rule entirely and set things up in the opposite direction like so:

```
class Square {
  float area();
  void setLength(float r);
}
class Rectangle extends Square {
  void setHeight(float r);
}
```

But this too drives us nuts. A `Square` is supposed to behave in a certain way, and `Rectangle` starts breaking those rules. This makes it strange to talk about a `Square` that might not actually be a square actually…

There are two sensible ways to deal with this problem. First, we could make these classes immutable. This gets rid of the state, which drops the invariance and makes things covariant again. Now there’s no issue with `Square` being a subclass of `Rectangle` (except that it might vex you there’s two member variables, `length` and `height`, storing the same value…)

But more likely, since we often reach for objects to model state, the actual answer is: *“is a” is a crock, and just don’t use inheritance at all.*

```
interface HasArea {
  float area();
}
class Square implements HasArea {
  void setLength(float r);
}
class Rectangle implements HasArea {
  void setLength(float r);
  void setHeight(float r);
}
```

What we like is dynamic dispatch. Figure out what interface you want to dispatch over, and write that down, and forget inheritance in your design entirely. Head scratching over whether “this ‘is a’ that” is largely a waste of time, and yields poor designs anyway.

### Liskov substitution principle

The way we reason about our code in the presence of classes is to have properties in mind about their behavior. For a given class, we might have some property *P* in mind, and so it should be true that for every instance of that class *o*, *P(o)* holds. At first, this is easy, because “every instance of that class” is just instances of the base class itself.

But once we start subclassing, we start introducing whole new, er, classes of instances. The general idea of the [Liskov substitution principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle) is that, as the author of a subclass, you have to ensure these properties are still true.

If you thought I was being clever above when I tried writing `Rectangle` as a subclass of `Square`, here’s where we immediately see how not-clever that actually was. With squares, we have properties we think should hold, like that every instance has area that’s *l²*. That becomes untrue of rectangles, and so this just isn’t a valid subclass at all. If we start breaking this rule, our design is terrible, and it’ll catch up to us eventually.

The trouble we suffer at the hands of inheritance here are:

1. We don’t actually think about and write down what these properties are. **Choice of these properties is a major design element for a base class.** So we’re often just totally giving up on reasoning, and dropping a design task on the floor.
2. All of this complexity of thinking about the properties can be **almost entirely avoided** by working with interfaces instead of a base class. Interfaces have properties too, but these are much simpler, more obvious, and less fraught with difficulties, even though we still don’t usually write them down.
3. Inheritance is often pitched as a tool for “re-use”, and as programmers we often get so absorbed in a problem that our immediate concern is whether “it works” or not. As a result, **this principle is routinely violated** because after deciding it works, we don’t think about whether it works *well*.

Anecdote: There’s more than a few examples of even rigorous academics making this mistake. At a functional programming conference, after presenting a paper involving a new type that was an instance of `Monad`, an attendee stood up and informed them their type violated the monad laws. (The monads laws basically being for that Haskell type class what our properties above are for base classes / interfaces. There’s a property we specify on the interface that every instance should obey.)

They fixed the problem, but were forced to remove a useful feature as a result. Some people might not believe that’s a worthwhile trade-off, but adherence to these kinds of reasoning principles is part of the reason functional programmers comment so often about how “if it compiles, it works!” As programmers, we’re so used to writing code and then having to debug it, that when that cycle gets cut short because there’s little to debug—because the code is so easy to reason about that fewer mistakes happened in the first place—it’s very surprising.

## How should we cope?

To summarize the above, inheritance can cause a few problems:

- We couple an interface (a type) with a specific implementation.
- We have difficulty evolving our code in the future, thanks to unrestricted open recursion.
- Part of the reason we like objects is to model state, but state and variance don’t play well together.
- We routinely punt on actually employing the proper reasoning tools necessary to make inheritance, well… reasonable.

So what should we do about it? Our languages are here to stay for the foreseeable future, and we still need to write programs today. As usual, I think there’s [two perspectives to take](https://www.tedinski.com/2018/01/30/the-one-ring-problem-abstraction-and-power.html).

From an “outside” perspective, as a user of classes others have designed, there isn’t much to say. Most of us have already heard “favor composition over inheritance” and from this perspective, all that means is not making the mistake of inheriting from something you could just as well have simply had as a member variable. This is a common mistake of new programmers, who have been taught a lot about inheritance and so think they’re supposed to go use it, but I’m not sure it a common problem among experienced programmers.

The interesting part is advising the designers of classes. Here, we benefit immensely from distinguishing between internal classes and classes we’re [exposing as part of a system boundary](https://www.tedinski.com/2018/02/06/system-boundaries.html).

- For internal classes, where one project contains the base and all subclasses, don’t sweat it. Here you can sometimes get away with actually treating inheritance like it is for “re-use.” There are still better and worse designs, of course, but in this situation, *simpler and smaller* are probably the biggest concerns, and you still have the ability to refactor things however you please in the future.
- For classes being exposed externally, there are two things we should always do:
  1. Here is where it is actually important to decouple interface and implementation. As your system evolves, you will eventually be concerned with introducing alternative implementations without breaking your users. (Do try to come up with better names than `IThing` and `ThingBase` though, please.)
  2. Here is the harder part of design: ideally, you would never put your users in the situation where they way they use something in your library is by inheriting from it. How to avoid this is not always an easy question to answer.

This essay is already way too long, but I do feel compelled to try to give some initial clue as to *how* one can avoid needing to inherit from a base class. So here is (in my opinion) an interesting tactic: after separating interface and implementation, attempt to factor out as much code as possible from your base class as public static methods. Ideally, your base class that implements some default behavior would have methods that are 1-3 lines of code, and largely just consist of calling the static method that actually does something. The reason this is beneficial is that all of these become individually re-usable pieces of code, decoupled from each other and from the base class. More on this another time.

#### Comments

[Comments section for Patreon backers](https://www.patreon.com/posts/16980221).

[Patreon](https://www.patreon.com/tedinski)

This post is part of a [book writing effort](https://www.tedinski.com/book/). Consider [supporting me on Patreon](https://www.patreon.com/tedinski).

#### Read more

- Follow me on Twitter: [@tedinski](https://www.twitter.com/tedinski/)
- Follow my [RSS Feed](https://www.tedinski.com/feed.xml)
- Check out the blog [archive](https://www.tedinski.com/archive/)
- Prev: [System boundaries: the focus of design](https://www.tedinski.com/2018/02/06/system-boundaries.html)
- Next: [What would OOP without inheritance look like?](https://www.tedinski.com/2018/02/20/an-oo-language-without-inheritance.html)

Copyright © 2018 [Ted Kaminski](https://www.tedinski.com/about/).