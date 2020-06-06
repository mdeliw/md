# Composition vs extension

Apr 16, 2019

I have written about the drawbacks of [inheritance](https://www.tedinski.com/2018/02/13/inheritance-modularity.html) in the past, and the notion that we should “prefer composition over inheritance” is well-known. But I’d like to step back a bit today, and contrast composition with *extension*, not inheritance specifically.

This is a bit of a muse day. I’m not entirely sure this is going anywhere fruitful, yet!

## The similarities and differences

If we think about things at the module-level, as I often like to do, then “extension” looks identical to mere “use.” Module A imports module B, and that’s all there is to either one of these situations. How should we understand the difference between “A extends B” and “A just uses B” or even “A is the composition of some new things with B.” Or is there even a difference?

I think there is, but I’m also not sure how to clearly state it. (Hence, today’s muse post.) Here are a few ideas about why I think there’s a difference:

- Inheritance clearly is a case of “extension” and not simple use. The trouble here is drawing a line between the problems caused by inheritance specifically, versus the drawbacks of “extension” in general.
- As I’ve argued in the past, [designing for extensibility has cost](https://www.tedinski.com/2018/08/07/what-makes-good-design.html), and we don’t always want to that. This all comes back around to [system boundaries](https://www.tedinski.com/2018/02/06/system-boundaries.html) and the desire to make things easier to *modify*.
- Could “extension” be as simple as implementing interface types in general? I’m not exactly sure. Perhaps this is the root of the trouble I’m having, but it seems to me there’s room for some kinds of new interface implementations that are just a “use” versus some kinds that are an “extension.” For example, function types are a kind of interface, but I’m not really sure it makes sense to think of every new function as an “extension.” This seems over-broad to me. So then what separates these two different cases?
- What is the contrast of extension? In other words, what does “mere use” even mean? Is it just “composition?” I’ve described in the past how [programming languages are inherently compositional](https://www.tedinski.com/2018/05/15/familiar-forms-of-composition.html). We build bigger functions by composing together smaller functions, and ditto statements and expressions. Is everything either extension or composition?

## Similar dichotomies

Something else that strikes me as similar is the difference between a “framework” and a “library.” Both are libraries, really, but somehow frameworks are more… encompassing. Obviously you can merely use a library, but can one “merely use” a framework? What’s the difference and why?

Likewise, when we’re building plug-ins for applications, we’re clearly extending that application’s behavior somehow. Is this something that just exists in our minds, or is there a clear way we can define how this **isn’t** composition?

Or if I write in an application’s scripting language? Maybe we can exploit the metaphor of a “runtime” to explain the difference? Maybe when we have a more-constrained runtime it looks like extension, whereas a more liberal runtime starts to look more like use?

## A story about my past work

One of the interesting things I did with my PhD research was develop what we called a “modular well-definedness analysis for attribute grammars,” which is way too much jargon for the uninitiated. Here’s the de-jargoned version:

The neato thing about attribute grammars (think: kinda neato object-orientedish, but not exactly, purely functional programming language) is that you can really separate out the parts of a program. To use OOish language, you can explode declaring an interface, declaring each method that’s a part of that interface, declaring types that implement that interface, and declaring each implementation of each method of an interface into separate files or modules if you want. Obviously, this is a recipe for disorganized disaster if you really go hog-wild with that.

So part of what we wanted was a way of repairing things back to sensibility. We reached for *modular reasoning* as the approach to accomplishing that. The language lets us put declarations *!@$#$!@ing anywhere*, but we are going to come back around and re-impose order. The over-arching rule is simple: you should be able to understand each module in terms of its imports, and without having to know all modules included in the whole program.

The end result was kinda nifty. I found some minimal rules for where declarations could reside, and as a result, the program kinda “condensed” around some root declarations, with only a few decision points about how to structure things.

So we can start with module H (the “host”) and have module E (the “extension”) depending on it. What’s the difference between “extension” and “mere use” here? Well, we can restructure this program into three modules: leave H unchanged, but separate E into two modules, G which depends on none of these modules, and C, which depends on both H and G. We do this by moving into G every declaration from E that’s not *forced* into C by the rules. In other words, we try to restructure the “extension” into a composition (C) of two modules, H and G.

The difference between an “extension” and a “composition” here was just one of degree. For “obvious compositions,” quite a lot of code could validly be moved from E to G, and the composition C was relatively light. For “definite extensions,” sometimes literally 100% of the code could not be moved into a separate module. It was all inseparable from H.

## An alternative hypothesis

I’m not sure the above story gives good intuition, though. I think it’s really cool, but there are parts of it that strikes me as potential coincidence. We might use the word “extension” for multiple distinct things. (Also, part of what my thesis work may have involved was a way to eliminate “extension” in favor of exclusively doing “composition,” and so the above might be misleading.)

So here’s my final thought on what the difference is:

**We’re composing things together when we can \*reason\* compositionally about the result, and we’re extending when we need non-compositional reasoning.**

Non-compositional reasoning is when you cannot understand something from its obvious parts. Mutable state is the obvious example. You can declare some mutable state and… have no idea what happens with it. To really understand it, you need to know all possible mutators of that state. But from looking at the state itself, you don’t necessarily know where those even are (unless the state is encapsulated). Mutations can happen from anything that imports that module, so we’re in serious trouble.

The thing about being able to write extensions is that someone else might be writing an extension too. You might think you can reason compositionally, until the system surprises you with its non-closed world. Plug-ins conflict all the time.

But importantly, **some interfaces can be wholly understood in their own right.** (Though, we might admit some “leakiness” to the abstraction.) Function types are a reasonable example: you can call them by supplying the right arguments. (Ah, but we might worry about side effects and such, but! That’s just the leaky part.)

How can we know everything we need to know? Well, I’ve showed one example already back when I discussed [property testing](https://www.tedinski.com/2018/04/24/design-and-property-tests.html). So next week (probably), I’ll think a bit more on (deep breath) category theory. Or maybe more specifically, what we can learn about programming from it.

## End notes

I asked a lot of questions today. Got any thoughts of your own?

- Has anyone seen this concept explicitly stated before: “encapsulation of mutable state restores compositional reasoning.”

#### Comments

[Comments section for Patreon backers](https://www.patreon.com/posts/26145583).

[Patreon](https://www.patreon.com/tedinski)

This post is part of a [book writing effort](https://www.tedinski.com/book/). Consider [supporting me on Patreon](https://www.patreon.com/tedinski).

#### Read more

- Follow me on Twitter: [@tedinski](https://www.twitter.com/tedinski/)
- Follow my [RSS Feed](https://www.tedinski.com/feed.xml)
- Check out the blog [archive](https://www.tedinski.com/archive/)
- Prev: [Controlling module dependencies](https://www.tedinski.com/2019/04/09/module-anti-dependencies.html)
- Next: [Encapsulating mutable state](https://www.tedinski.com/2019/04/30/encapsulating-state.html)

Copyright © 2019 [Ted Kaminski](https://www.tedinski.com/about/).