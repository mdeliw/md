# Data, objects, and how we're railroaded into poor design

Jan 23, 2018

I don’t think we have any actually *good* programming languages, and I don’t think I’m alone in believing this. Programming is hard, and language design is harder. We’re still learning. But I think they’re all failing us in a shockingly fundamental way.

The root of the trouble is a distinction I’d like to draw between **data** and **objects**. Let me know if you think there are better terms to use.

Programming languages give us tools to represent things. Sometimes these things are values: the integer 1. A 1 is a 1, same as any other 1. Sometimes these things have identity: *this* integer 1 lives over here, and *that* integer 1 lives over there, they aren’t the same. This can matter because (just as one example) sometimes these things are mutable: now *that* integer 1 becomes integer 2. It wasn’t just a value.

For some things, the internals should be hidden away, but for others, we wonder why we’re expected to litter all these repetitive getters and setters about. Sometimes, we can use a thing here or there without issue, and sometimes we have to *serialize* and *deserialize* it to get around. That action causes it to lose some of its identity: now there are two things.

Sometimes we care about how these things get used, especially how they get extended. Perhaps we decide a `Thing` has a fixed set of behaviors. (I’m going to stop playing coy: an interface, with a fixed set of methods.) Then users can end up creating their own `Things` (i.e. new classes that implement that interface). Or alternatively, we can decide a `Thing` has a fixed set of variants (i.e. algebraic datatypes). Then users can freely create any behavior—any function—over a `Thing`.

If you’ve been counting, that’s 5 different choices a language can make about how to represent something. Naively, there could be 32 different possible designs, and that’s assuming there aren’t a lot more choices than the ones above (and that they’re all binary… some aren’t!) But these decisions correlate together into essentially just two major designs that make a lot of sense:

|               | Data                                                         | Objects                                                      |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Equality      | Values: any 1 is a 1.                                        | Object identity: *this* 1 is not *that* 1.                   |
| Identity      | It’s just bytes, copy away.                                  | You can serialize / deserialize or clone, but that’s an *action* that gives you a new object with distinct identity. |
| Mutability    | Immutable: nothing else makes sense. You wouldn’t assign `1 = 2`. | Usually mutable: objects help us organize state.             |
| Abstraction   | Exposed internals: it’s just data, it conforms to a schema. There’s no harm in seeing it. | **Encapsulation** is one of the best aspects of OO, you can maintain internal invariants, as long as you don’t mistakenly expose mutable internals. |
| Extensibility | The schema gives us a fixed set of variants, over which you can of course write any function you want. | We have a fixed set of exposed operations, but different variants can be constructed (including an improved ability to evolve a variant without impacting clients). |

We do see a couple other variants in the wild, but I submit that (other than a few domain-specific examples) these are really just work-arounds for inadequate language design. We have immutable objects so often in Java because, *what choice do you have?* You can’t actually represent data as *data* in that language. Java’s support for data begins at `int` and ends at `float`. C doesn’t do much better: tack on `structs` and arrays.

An anecdote: ever see generated code in Java try to store a blob of data? Say, an array of integers? If you just have a static array, well, that turns into bytecode that allocates an array and one-by-one assigns each element to the correct value at class load time, and can quickly exceed the bytecode size limits for a method. There isn’t (yet?) a format for just any kind of data in `.class` files, so you end up encoding the data as strings and then concatenate and decode that string at runtime.

*“We built this nice compact state machine transition table and now we… oh, **EWW**!”*

In a world where we’re good at program design, we’d be making a conscious choice about whether the thing we’re trying to represent is *data* or an *object*. And our programming languages would help us and support us in representing it as we chose.

Hahahah.

![img](https://www.tedinski.com/assets/visualized-representational-paradise.svg)

An attempt to visualize my ideas about how our languages handle representations. There are two alternative paths to supporting both objects and data: clearly distinguish between then, or conflate them.

### The objects extreme (e.g. Java)

When I [pitched this book idea](https://www.tedinski.com/book/) I made a few disparaging comments about object-oriented programming. My biggest criticism can be summed up by looking at what I’ve written above and then pondering the slogan “Everything is an object!”

**Data makes sense**. We want data, a lot. We work with data, a lot. I think we have a (possibly under-recognized) preference for data. And yet we have many languages designed with “everything is an object” in mind.

People criticize Java for having `int` and friends, and say this is evidence that Java didn’t go far enough in making everything an object. (Especially in light of the need for `Integer`.) But I think the more accurate criticism is the other way around: Java didn’t go far enough in having support for actual data. (Not that primitive types don’t cause some problems with things as-is, of course. And the JVM folks are working on a “value types” proposal, which I am excited about it especially in that I hope we learn a lot about language design from it, but I also suspect backwards compatibility will always make this at best a work-around design.)

We still manage to do data-like things with these languages, of course. Making more objects immutable is a (probably deserving) fad right now in Java circles. People are often thrilled to work with `map`, `filter`, `fold`/`reduce`, `flatMap`/`collect`, on streams of *data*. Or even streams of “let’s try our best to mimic data.”

But many immutable objects are just things we wish were data instead. (Not all, however: there are “abstract data types” that can make sense as an immutable object wrapping data, but perhaps this is a good topic for another article…)

### The functional extreme (e.g. Haskell)

```
data JSValue =
    JSNull
  | JSBool  Bool
  | JSNum  Double
  | JSString  String
  | JSArray  [JSValue]
  | JSObject  [(String, JSValue)]
```

All there is to representing JSON, in algebraic data type form. “A `JSValue` can be one of six things…”

One of the most important kinds of data are **trees**, and the ability to represent trees as data is sorely lacking in nearly all OO languages. Algebraic datatypes are one of the key tools to do exactly that, and functional languages are often built around algebraic datatypes like OO languages are built around objects. So they naturally manage to do this better.

But many functional languages eschew objects, and are often skeptical of state generally (which I think is at least a little bit bound up with objects.) Haskell in particular also manages to be rather lacking in representing records, arrays, and strings. It’s still an amazingly good research platform, its drawbacks are minor for many applications, and being forced to think without objects is an exercise I think every programmer should go through.

A notable feature Haskell does have are typeclasses. These can serve as a replacement for certain things one might reach for an interface for in an OO language, but still, they are entirely non-replacements for objects. Indeed, even attempting to simulate objects will get Haskellers upset with you. And probably for good reason, given the way things are.

Also notable are Standard ML’s structure/functor/signature-based module system. Modules are arguably better than objects, for some uses. For example, with OO languages, there’s sometimes a kind of “start up” phase where an object graph is constructed that represents the configuration of the system. OO programmers often reach for “dependency injection” frameworks to help work with this, as constructing and threading around all the state can be irritating without one. With SML, all this can be accomplished in a much more principled way with a very interesting, and somewhat object-like, module system. But like Haskell’s typeclasses, this isn’t really a replacement for objects generally, just some uses of them.

*An Aside:* I’m staking out a position that claims objects are useful. Certainly, there are some who believe ML modules are wholly and entirely superior to objects. I’m not entirely sure my point of disagreement with them is significantly deeper than definitions of terms, however. For instance, I’m also a bit skeptical of inheritance, which I’ll write more about later. One might reasonably claim that objects without inheritance are just crummy ML modules, but I think there are ancillary design differences with objects that are actually quite important. So for now I’m going to stick to “objects are good and we want them” and claim that modules aren’t good enough. Getting to the weeds on why I think so might be an interesting topic someday, maybe. Oh, and even though I’m including SML here as an example of “the functional extreme,” it probably has the most potential of any current language to evolve in a good direction.

### The path of conflation (e.g. Scala, C++)

Naturally, one might immediately try to simply put all these features together to get the best of both worlds, no? Unfortunately, there seem to be several traps that prevent this from being a simple task.

One trap is to simply jumble the features together with no real coherent though put into how things should be used. C++ rather famously bills itself a multi-paradigm language, but I’m not sure anyone really thinks of it that way. We do think of it as the undisputed monarch of complexity, however.

Another trap is to design something different but use similar terminology. Every object is an instance of a class: simple, right? Well, OCaml has very different ideas. And not all good ones, considering the general advice about the object part of the language is “[don’t use objects](https://stackoverflow.com/questions/10779283/when-should-objects-be-used-in-ocaml).”

Another trap is to search for one unifying super-abstraction that subsumes all else. Scala is guilty of this one, as it’s quite explicit (and has to be, to understand what’s going on) that its “case classes” are just syntactic sugar for a bunch of classes, and don’t much behave like data otherwise at all. Which is understandable, since it’s on the JVM, and so has the same underlying availability of semantics as Java. At least it offers pattern matching.

### Representational paradise (or, let’s chat about Erlang)

Since I’ve taken issue with just putting things together, what does an *actually good* design look like? Is there a language that truly supports us in taking a better approach?

Well, sort of. Erlang is often described as a functional programming language, but oddly enough it has one of the best ideas around for how to go about doing object-orient programming.

It’s just… it also conflates objects with its own notion of processes, which are like light/green threads or actors. Not a decision I’d recommend future languages make (if only because Erlang already occupies that niche, so find a way to do better! Well, and also because I think `async`/`await` is one of the biggest recent breakthrough in PL design, but I digress…)

Don’t know Erlang? Don’t worry! Check out the first two sections of [this chapter from *Learn You Some Erlang*](http://learnyousomeerlang.com/more-on-multiprocessing).

The key takeaway here is that `fridge` becomes an object-like thing (an Erlang process) which can be sent messages, in this case to `store` or `take` items in the `fridge`. Despite Erlang basically working only with immutable data, a process (from a broader system-level perspective) essentially has its own mutable encapsulated state.

The key thing that Erlang gets right is that processes (objects) can be sent data, and reply with data. Functions take data as input and produce data as output. So the default mode of operation is working with data. Quite rich data, that behaves like actual *data*. And yet, the overall system behaves like a graph of objects with what is in effect mutable internal state. Objects that are encouraged to exchange sophisticated data.

This is much closer to the ideal that I have in mind.

## We can see this affect our designs

This industry recently suffered a large NoSQL fad that—thankfully—seems to be receding. I would like to offer this hypothesis for why this fad happened in the first place: it could store data. Traditional RDBMS have tables that act as arrays of structs of primitives. There is no sensible way to represent tree-like data in that environment. I don’t blame anyone for a little irrational exuberance after being freed from that constraint. It’s so liberating it has to be something other than a database, right? “Document store!” Wow. What we now see is a smaller fad of blog posts pointing out that Postgres with a JSONB column is functionally superior in basically every respect in comparison to a NoSQL database. Ah, well.

Speaking of JSON, I suspect a large part of the reason REST has taken over the industry is because it, too, lets us exchange actual data. HTTP endpoints act like mutable objects that can be sent JSON messages (actual data!) and respond with JSON messages. It’s a very nice style of programming. There are arguably a lot of advantages to things like SOAP/WSDL: schemas, for example. But I think many of these systems succumbed to thinking in terms of passing around serialized objects, and so REST/JSON came with a serious advantage, despite its other drawbacks.

One of the classic errors in service architecture design is inappropriate use of internal “entity services.” It’s pretty much a service that just does CRUD for other services. Consultants have been trying to convince programmers to cut this out for over a decade, but it just keeps coming back. [I’m going to link to this interesting example, because I think it’s illustrative.](http://www.michaelnygard.com/blog/2018/01/services-by-lifecycle/) Fundamentally, the issue I see in the example given is that the poor design results from thinking of `Loans` as objects. If it’s an object, it has to have identity, and if it has identity, someone has to store it. The improved design comes from thinking of `Loans` as data, to be passed around between objects. `Loans` no longer have their own identities, except perhaps internally with each service. Regardless, we get a better design by distinguishing between `Loans` as data and `Loans` as objects.

## Embracing the distinction

I think if we’re to get better at design, we need to be making a conscious and explicit choice about whether we’re representing data or objects. In the long term, I think we’ll want languages that do a better job of encouraging and allowing us to make this decision. They should do a better job of supporting *both* design choices distinctly, without conflating them together.

In the nearer term, we should be extra careful to ensure we’re not accidentally making this decision without thinking about it. After all, communities surrounding a language often express strong cultural preferences, to the point where it’s sometimes not obvious a choice is even being made. Having recognized what we actually want, we can apply whatever workaround makes the most sense, even if the language isn’t directly helping us. Whatever we’re stuck with: immutables objects, getters/setters, the visitor pattern, boilerplate code for comparison, hashing, cloning, serialization/deserialization, and so on.

I’ll have more to say on this later, but I’d be interested to see what you think.

#### Comments

[Comments section for Patreon backers](https://www.patreon.com/posts/16563453).

[Patreon](https://www.patreon.com/tedinski)

This post is part of a [book writing effort](https://www.tedinski.com/book/). Consider [supporting me on Patreon](https://www.patreon.com/tedinski).

#### Read more

- Follow me on Twitter: [@tedinski](https://www.twitter.com/tedinski/)
- Follow my [RSS Feed](https://www.tedinski.com/feed.xml)
- Check out the blog [archive](https://www.tedinski.com/archive/)
- Prev: [How humans write programs](https://www.tedinski.com/2018/01/16/how-humans-write-programs.html)
- Next: [The one ring problem: abstraction and our quest for power](https://www.tedinski.com/2018/01/30/the-one-ring-problem-abstraction-and-power.html)

#### External Links

I noticed these discussions of this post: [Reddit](https://www.reddit.com/r/programming/comments/7sf35i/data_objects_and_how_were_railroaded_into_poor/)

Copyright © 2018 [Ted Kaminski](https://www.tedinski.com/about/).