# Notes on the expression problem and type design

Mar 6, 2018

Last week, I brought up [the effect the expression problem has on type design](https://www.tedinski.com/2018/02/27/the-expression-problem.html). (Although I think this week’s post is readable without having read that one, you might still want to if you haven’t already.) To recap, there’s three simple, fundamental ways we can choose to design our types: as data, as objects, or as abstract data types. These three design options affect the extensibility (in terms of new variants and operations) of the type: either by users of our module, or by ourselves in future versions of the module (without breaking its users).

There were several important things I didn’t get into, though, either because the post was long enough as-is, or because I didn’t think of it at the time, and got feedback after posting. Thanks! Let’s talk about that.

## What *exactly* is the expression problem?

While I think I gave most people the gist of it in the last post, I neglected to be precise. I do think the gist is the important part—the problem certainly has its origins in looking at the extensibility properties of objects vs algebraic data types and wondering “why can’t we have our cake and eat it too?”

So more precisely, the expression problem is an attempt to design an abstraction mechanism for a programming language that meets the following requirements:

1. **The type is openly extensible with new variants.** Just as with objects, where once you have defined an interface type, anyone can freely define new classes implementing that interface without causing anyone else any problems.
2. **The type is openly extensible with new operations.** Just as with algebraic data types, where once you defined the fixed set of constructors, anyone can freely define new functions that pattern match on that type without causing anyone else any problems.
3. **Strong static type safety must be possible.** See below for some discussion on dynamic languages, but there nevertheless needs to be some way to reason about what’s going on with your type, especially where “missing method implementations” are concerned. Static type safety is a reasonable way to ensure this is not only possible, but can be automated in the form of a type checker. Our definition of “strong” in this case means no reflection or `instanceof` or the like; we want the type system to do all the reasoning for us.
4. **Without modifying or duplicating code.** To some extent this should go without saying, but you have to cut people off from cheating in more ways. “Of course I can add new methods to an interface! Watch me edit its definition.” is not an acceptable solution.
5. **With separate compilation and modular type checking.** The reasoning method we had for criteria (3) should be modular, i.e. not reasoning about the whole program. So for a static type checker, the error checks have to be for our module, not for a later linking phase, “weaving” phase, or at runtime.

This particular definition of the expression problem, while commonplace, actually has an odd hole in it. A perfectly conforming solution could be one where it is an error to import multiple extensions to the same type. This is odd behavior from a programming language—we expect imports to maybe gives us name clashes, but not to just break our programs. To fix this problem and keep things sensible, we have to solve what got termed “the independent extensibility problem” ([pdf: Zenger, Odersky](https://infoscience.epfl.ch/record/52625/files/IC_TECH_REPORT_200433.pdf;)). To do that, we also have to meet what I like to call the composition criteria:

- **We must be able to compose independently developed extensions to the type.** In particular, we have to be able to import one module that adds a new variant, and another module that adds a new operation, and our solution for criteria 3 should inform us that we need to provide an implementation of that new operation on that new variant.

So there we go: open extensibility in both directions, subject to some rules to keep things (supposedly) sensible. Or at least, type correct.

I’m perfectly happy to see dynamic languages admitted to the expression problem club. The trouble with dynamic languages is that it’s easy to do preposterously awful things and then claim it’s a solution. The canonical example of this is [monkey patching](https://en.wikipedia.org/wiki/Monkey_patch). What behavior are you going to get from anything you import? Well, that depends on what might get imported by anything else in the currently running interpreter instance… so, who knows?

But I think dynamic languages can do this in reasonable ways as well. All they need to do is take reasoning a little bit seriously. We’ll take a look at Clojure in a bit.

## Terminology, revisited

Or, “another reason I’m writing this book.”

Last week, I made note of using the term “abstract data type” in a slightly different manner than usual. But really, perhaps I should have gone into the terminology a bit more in depth.

I think the design choice we’re presented with by the expression problem is a truly fundamental one. We model things with types, and you can’t design a type in any language without deciding how it’s open or closed to extension.

*And we have no common vocabulary for talking about these concepts?* Somebody tell me if I’m missing something, please.

![img](/Volumes/storage/Documents/md/assets/expression-problem-20200502084827921.svg)

Two dimensions of extensibility for a type: variants and operations.

I’ve settled on a few choices. The least controversial, probably, is **data** to describe types that are extensible in new operations. For one thing, the only other good candidate name I can think of is “values.” But at least “data” is commonly used. File systems store data, and what are file formats but another kind of data type? Databases store, er, data, and what are schemas but another kind of data type?

The next most controversial is probably **abstract data type**. The origins of this term were certainly meant to contain the notion of a class from object oriented programming. But… for one thing, I needed *some* term to describe the “closed to extensions” design space. And “abstract data type” extending to cover objects is just another term for, well, objects. And that term is in much more widespread use. So hopefully people will play along with my redefinition, or I’ll get enough blowback soon enough that I can decide on “closed types” or something else.

But perhaps the most controversial (and I didn’t even realize it) was using **objects** to describe the “extensible with new variants” design space. “After all,” say many advocates for non-OO languages, “don’t we have something else to offer here?”

### Let’s look at type classes

Last week, I described a simple type consisting of two variants and a single operation. Here we model that using type classes in Haskell:

```
class Coord a where
  distance :: a -> Float

data Cartesian = Cartesian { x :: Float, y :: Float }
instance Coord Cartesian where
  distance (Cartesian { x, y }) = sqrt(x*x + y*y)

data Polar = Polar { angle :: Float, dist :: Float }
instance Coord Polar where
  distance (Polar { angle, dist }) = dist
```

This certainly seems to fit the bill, right? We certainly can introduce new types that can call them instances of `Coord` freely, so why am I choosing a name like “objects” when we have non-object designs like this one here?

Well, let me show you a simple thing we can do with the original data-variant of the design from last week:

```
points = [Cartesian { x = 3, y = 2 }, Polar { angle = 30, dist = 1 }]
```

And here’s some simple code showing you that same thing with the object-variant:

```
Coord[] points = { new Cartesian(3, 2), new Polar(30, 1) };
```

And here’s what we have to do with type classes:

```
data AnyCoord where
  ToCoord :: Coord a => a -> AnyCoord

instance Coord AnyCoord where
  distance (ToCoord x) = distance x

a = Cartesian { x = 3, y = 2 }
b = Polar { angle = 30, dist = 1 }
coords :: [AnyCoord]
coords = [ToCoord a, ToCoord b]
```

This isn’t the prettiest code. Part of the trouble we’re having is that **`Coord` isn’t even a type** in the type class variant of this design. Since I’m describing points in the design space for types, type classes aren’t even eligible!

The design I show above is just one workaround to the problem. This workaround is [regarded as an “anti-pattern” by some Haskellers](https://lukepalmer.wordpress.com/2010/01/24/haskell-antipattern-existential-typeclass/). But this view isn’t really universally held, and you see libraries like the [Gtk bindings for Haskell](https://github.com/haskell-gi/haskell-gi) pretty much adopt this approach. (To see a direct example, [here](https://hackage.haskell.org/package/gi-gtk-3.0.19/docs/GI-Gtk-Objects-Widget.html) is the documentation where you can see a `Widget` type that analogously wraps a `IsWidget` type class, with the necessary `toWidget` cast function to be able to upcast from specific subclasses.)

*Aside: Could Haskell fix this problem?*

Short answer: yes, but there’s a reason they didn’t. It starts off looking easy enough, why can’t we just automatically generate that boilerplate? Then we could write:

```
coords :: [Coord]
coords = [a, b]
```

But this runs into a problem: our type system now needs subtyping relationships. We need to know not just that `a` has type `Cartesian` but also that `Cartesian` is a subtype of `Coord`. Subtyping is a horror show for type system design.

- Type systems with subtyping are dramatically more complicated than those without. [After 12 years, some clever people figured out an unsoundness bug with the Java type system](http://io.livecode.ch/learn/namin/unsound) that no one else saw. Oops.
- Type systems with subtyping do worse at inferring types than those without. It’s only relatively recently that Haskell’s design aesthetic is starting to embrace the idea that we can require some type signatures if the types’ presence will “do useful work” for us.
- All this complexity and all you’re getting is the ability to leave out `ToCoord` in a couple of places? Is that worth it?

Personally, I suspect the answer is “yes,” but it’s not a question easily answered. Someone has to figure out how to tame all that complexity.

### What about ML modules?

We can port over the type class design to work with Standard ML signatures/structures easily enough:

```
signature Coord = sig
  type coord
  val distance : coord -> real
end

structure Cartesian : Coord = struct
  type coord = { x : real, y : real }
  
  fun distance ({x, y}) = Math.sqrt(x*x + y*y)
end

structure Polar : Coord = struct
  type coord = { angle : real, dist : real }
  
  fun distance ({angle, dist}) = dist
end
```

Signatures are just types for modules (structures), so we do actually have a type here. But now we actually encounter an even bigger problem: there’s literally no way to have a list of “anything that meets the signature `Coord`.” None at all.

Standard ML attempts to cope with type system complexity by partitioning the language into two: there’s the module language, and the term language. The module language has all the machinery we’re interested in, but the term language does not. And there’s a stratification: we can go from the module language to the term language, but not vice versa. This missing feature (“first class modules”) makes this kind of abstraction, using signatures, impossible. We can’t have a list of arbitrary `Coord`s.

Ocaml has first class modules, but the code I wrote to demonstrate it is so awful looking, I’d rather not post it. On the other hand, the “o” in Ocaml is a language feature that does support interface types in the fashion we’re looking for, but that’s another story.

### …and that’s why I call the region “objects”

Type abstraction is important. When we’re modeling things in our programs, we’re designing types, even in dynamically-typed languages.

The name “objects” gets tangled up with a lot of other things. Some are likely serious misdesigns (e.g. [implementation inheritance / open recursion, which you can read about in a previous post of mine](https://www.tedinski.com/2018/02/13/inheritance-modularity.html)). But this is one area where I think objects got it very right. Objects are types—real, ordinary, first-class types, usable and *composable* like any other type—that are open to extension with new variants and have a fixed interface.

*Aside: So does that mean objects are the best?*

Not necessarily. We don’t have to give in to the impulse to just unify everything into one abstraction mechanism, just because they accomplish similar goals. Type classes offer ways to gives instances for type you don’t control, which interfaces typically don’t. Likewise, type classes can be inferred and be generated by what are essentially logic programming rules. ML modules can describe relationships between types that are difficult for interfaces.

The programming language of the future might have both objects and type classes, with some thought given to how they interact, or it might have some combined design that does both at the same time. Or it might disaggregate features into a more sensible design we haven’t thought of yet.

## “Told you so” -Cassandra, probably

Awhile back I wrote about how I think human programmers have this innate cognitive bias [to prefer *more powerful* abstractions and ignore the costs](https://www.tedinski.com/2018/01/30/the-one-ring-problem-abstraction-and-power.html). The immediate feedback from some readers was, of course, to ignore the costs and immediately sing the praises of their favorite powerful designs.

I don’t know what I expected.

So of course last week I bring up the expression “problem,” but also dismiss solving it as being too complex to be generally worth it. Can you guess what happened next?

Okay, I’ll stop with the snark. It’s on me to start stacking up examples to try to convince you. I didn’t show you a solution to the expression problem. Let’s take a look at one, and try to see concretely where the complexity comes from.

### A base type

This example is partially adapted from [Eli Bendersky’s blog](https://eli.thegreenplace.net/2016/the-expression-problem-and-its-solutions/) which helpfully comes very close to illustrating a point I want to make. We’ll implement things using Clojure.

Suppose we want to model a mini-language. We’ll start out with addition, literals, and evaluation, and then add some things from there. Let’s start defining a couple things.

```
(defrecord Constant [value])
(defrecord BinaryPlus [lhs rhs])

(defprotocol Evaluatable
  (evaluate [this]))
  
(extend-protocol Evaluatable
  Constant
    (evaluate [this] (:value this))
  BinaryPlus
    (evaluate [this] (+ (evaluate (:lhs this)) (evaluate (:rhs this)))))
```

With a dynamic language, I don’t have a need to give a name to my tree type. But I do define two named record types that it currently consists of: `Constant` which just wraps a number (`value`), and `BinaryPlus` which wraps two sub-trees (`lhs`, `rhs`).

Next we declare a new protocol that consists of a single function, `evaluate`, and I then give definitions for this function on the two kinds of tree nodes we have so far. We can run a quick test:

```
(def value (BinaryPlus. (BinaryPlus. (Constant. 1) (Constant. 2)) (Constant. 3)))

(println (evaluate value))
```

And this gives us `6`. This is a simple, complete module. We can test it out, see it works fine on its own. This code is so simple, you can convince yourself it’s correct just by looking at it for a bit.

### A new operation

Let’s add the ability to render our trees into strings.

```
(defprotocol Stringable
  (stringify [this]))

(extend-protocol Stringable
  Constant
    (stringify  [this] (str (:value this)))
  BinaryPlus
    (stringify  [this] (str (stringify (:lhs this))
                            " + "
                            (stringify (:rhs this)))))
```

Like the base module, this is a simple, complete module. We know what we need to test: make sure this printing actually works, and it does. `(stringify value)` gives us `1 + 2 + 3`. We need to know about the module we’re importing (“What’s `Constant` and `BinaryPlus` again?”), but other than that, this module is also something simple enough that we can convince ourselves it’s correct just by looking at it.

### A new variant

Let’s add multiplication to our mini-expression language.

```
(defrecord BinaryMultiply [lhs rhs])

(extend-type BinaryMultiply
  Evaluatable
    (evaluate [this] (* (evaluate (:lhs this)) (evaluate (:rhs this)))))
```

Once again, this module, on its own, is easily testable. Because of the order I’ve presented things in, you’re probably also wondering about how to turn this into a string, but on its own this module is complete.

### The composition of these two extension modules

Putting these two modules together, we can fix up the composition. In a fourth module, we can supply the “glue code” we need to compose these two independent extensions:

```
(extend-type BinaryMultiply
  Stringable
    (stringify  [this] (str (stringify (:lhs this))
                            " * "
                            (stringify (:rhs this)))))
```

Composition problem solved. We can now happily use both `BinaryMultiply` and `Stringable` without any issues with missing implementations. It’s a little unfortunate that this code is scattered a bit, but as long as the number of extensions doesn’t get too large, this seems manageable. So everything is fine, right?

### Did you spot the bug?

What we’re actually suffering from here is a similar problem as the [“fragile base class problem” that plagues implementation inheritance](https://www.tedinski.com/2018/02/13/inheritance-modularity.html). What we want is *modular reasoning*. We want to look at our code, and understand it on its own terms. We want to be able to forget about all the other code out there, because that’s too much to fit inside our own head at once. We want to be able to convince ourselves our module is correct, so we have the ability to stop thinking about this module’s details, and we can go look for a bug elsewhere. That’s basic abstraction. And this design doesn’t let us do it.

The problem, if you didn’t spot it yet, is that while the `Stringable` module is *completely correct on its own*, it suddenly gets retconned into being incorrect when multiplication is introduced. A tree representing `(1 + 2) * 3` gets printed as `1 + 2 * 3` which is likely parsed as `1 + (2 * 3)`, a totally different expression. Parentheses are unnecessary to print when you only have addition, but that all changed when we didn’t only have addition.

What can we do to fix this problem? Well you have two options:

1. We can, in our composition module, override the implementation of `stringify` on `BinaryAdd` to add in parentheses. This works, but is no better than monkey patching. We’re mutating the behavior out from underneath other modules. Goodbye modular reasoning.
2. We can modify the original module introducing `stringify` to include parentheses, and add a comment explaining that elsewhere are reasons why this is necessary. But this too compromises modular reasoning. We can no longer just think about this module in isolation, we have to start thinking about all these outside considerations. What else might be “wrong” about our `Stringable` module because of some unknown other extension?

Further, we still have another fundamental reasoning problem: how are we supposed to catch problems like his? Some piece of code can be written, tested, and we can conclude it’s correct, and then suddenly it’s not actually correct, and there’s no obviously necessary test we can write in the module with the bug that reveals that bug!

The added reasoning burden we take on here is that a module composing multiple extensions like this *takes on the reasoning burden of all the modules it is composing*. While we’re technically able to re-use the code written in other modules when we combine the `Stringable` and `BinaryMultiply` modules, we cannot re-use any testing or thinking that was done about how that code works. We pretty much have to re-do it all, to be sure everything is still correct.

And the thinking is the hard part. If we just wanted to re-use code, why not copy & paste everything into one file?

*Aside: Can’t a type checker save us? What happened to the reasoning parts of the expression problem?*

Well, no, not really. Although this example is in a dynamic language like Clojure, this reasoning problem happens with **any** solution to the expression problem. The Clojure code I wrote is perfectly principled. Ordinary type checkers would be helping us spot a missing implementation, but not that the string we’re constructing is now wrong.

In fact, I mentioned last week that the expression problem is fundamental, and applies to mathematics, too. Well, even if you started thinking in terms of formal verification and theorem proving here, you just run into exactly the reasoning problem I mention. The composing module will discover the imported definitions and proofs have missing cases. (Just like the Clojure protocol has a missing case we had to supply!) You get the added responsibility of going back, understanding all those proofs, and filling in the missing cases, in order to make the composition work.

If those cases can be filled it, that is. Because the theorems might now be *wrong*.

## Are you sure you wanted a solution?

As a final note on the expression problem, there are common things languages will not allow that can make it seem like you need a solution, but you don’t really. These are things that can look a little bit like monkey patching at first, but it’s actually a perfectly reasonable and safe thing to do.

C# has a feature called [extension methods](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods) that allow you to remotely add new methods to a type. At first this sounds like it might be a problem, but these are statically dispatched methods. And unless you import the extension methods, you don’t see them and they have no effect. None of the problems with monkey patching arise. (I’m not completely certain, but I think [Swift protocol extensions](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Extensions.html) work similarly.)

To some extent, you can also think of Haskell type classes in this way, too. You can define a new type class, and give instances of it for data types from other modules. But Haskell also has rules about [orphaned instances](https://wiki.haskell.org/Orphan_instance) that keep this sensible. Without these restrictions, you might start to have multiple (different, conflicting?) instances for a type. So this, too, is safe. We do not get surprised with any horrible reasoning problems. And [Rust traits](https://doc.rust-lang.org/1.8.0/book/traits.html) work in a similar way.

So if we’re to try to develop a type that’s open to extension along both dimensions, we have to first make sure that’s really what we want. You might find you can get along just fine without it, and the price you would have paid may have been far higher than it was worth.

#### Comments

[Comments section for Patreon backers](https://www.patreon.com/posts/17401954).

[Patreon](https://www.patreon.com/tedinski)

This post is part of a [book writing effort](https://www.tedinski.com/book/). Consider [supporting me on Patreon](https://www.patreon.com/tedinski).

#### Read more

- Follow me on Twitter: [@tedinski](https://www.twitter.com/tedinski/)
- Follow my [RSS Feed](https://www.tedinski.com/feed.xml)
- Check out the blog [archive](https://www.tedinski.com/archive/)
- Prev: [Design duality and the expression problem](https://www.tedinski.com/2018/02/27/the-expression-problem.html)
- Next: [What can we learn from how compilers are designed?](https://www.tedinski.com/2018/03/13/how-compilers-are-designed.html)

Copyright © 2018 [Ted Kaminski](https://www.tedinski.com/about/).