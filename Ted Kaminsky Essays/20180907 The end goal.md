# What does it mean to design software well?

Aug 7, 2018

When we talk about design—as *good* design versus *bad* design—what are we even evaluating?

Sometimes questions like this are seemingly far removed from reality, or seen as “too philosophical” to be useful. But this is a good question to actually have an answer for, and if you haven’t thought about it, you probably should. So let’s take a short moment to think.

## The end goal

“[Look, kids, we’re in this for the money.](http://geepawhill.org/were-in-this-for-the-money/)”

Ok, so this perspective might be too “living under capitalism,” but still… most software is developed for money. We want to get paid. And we don’t get paid to make something aesthetically pleasing, we get paid to get things done.

Even when disentangling this idea from economic systems, we still end up in a very similar place. No matter what, we want to accomplish things quickly. Programming is really about the ends, less the journey. We want to *do* something, even if that something is “create a performance art piece raging against the idea that programming is only about the ends.”

Many of us got into programming because we found a way to have fun doing it, early on. My first lines of code were cheating at Apple IIe games. To me, programming seemed to be the closest thing to wizardry that actually existed. Magic is magical because it does seemingly impossible things, but most impossible things (that don’t just violate the laws of physics) are just ordinary things done impossibly fast.

The faster we can do the thing, the better. But there are some hard lessons to learn here, because we know there’s almost always more than just “the thing.” We also know we aren’t always right about what “the thing” is when we start out.

So even if the one major end goal is writing software quickly, we know there are reasons to take our time anyway:

- We want to learn things we didn’t know (and need to) or discover what things we’re wrong about, quickly.
- We want to get confidence the software works correctly, quickly.
- We want to be able to fix our mistakes, quickly.
- We want to wrap our head around other people’s software, quickly.
- We want other people to wrap their heads around our software, quickly. (Especially when “other people” is “me, two months from now.”)
- We want to re-purpose our software, as our goals change, quickly.
- When we make software that’s too slow, we want to make it faster, quickly.

If we ignore these concerns, we risk working hard and not smart. We pursue speed in a way that slows us down. In most non-trivial software, all these concerns come up eventually. It’s a false economy to ignore them. So we should plan for these from the beginning.

But the point underneath all this is simple enough: we need to think about what our goals actually are, and then how we’re actually going to achieve them. And we need to be willing to [loop back around](https://www.tedinski.com/2018/01/16/how-humans-write-programs.html) and re-think our goals when we learn things in the process that affect them.

Everything else isn’t really adding more to this underlying process, it’s just a tool to be more mindful of what our goals actually are. It’s hard to actually improve something if you’re not willing to look specifically at the problem and instead stick to vague generalities. If “keep it simple, stupid” were all it takes, there’d be nothing to learn here. It’s more complex than that.

## Some goals are inherently at odds

And it doesn’t help that we sometimes have to make trade-offs. If we have to build things fast, that’s pretty much inherently at odds with taking the time to be careful and do things right from the beginning. If we’re just going fast, it’s easier to go faster (right now) by skipping documentation, right?

But these don’t have to be as opposing as it seems. It just means we need to be careful about what parts can be done fast, and what parts should be done slower and more carefully. Even then, most design is iterative. Even if we do something quick and dirty, if we’re aware of its flaws, we can come back around and fix them before it’s too late.

Most of the best design advice manages to hit multiple goals at once. In that article I linked earlier ([“in it for the money”](http://geepawhill.org/were-in-this-for-the-money/)), test-driven development is being advocated because it *both* helps write code now, and helps maintain code later. I take issue with some aspects of what gets pitched as TDD, but I don’t really disagree with this general claim. Done well, you’re just taking a tiny bit of time to automate a testing process you have to do anyway. That effort can pay off very, very quickly. We’re not talking about a huge up-front cost that takes a long time to amortize into an advantage. It’s a modest effort that pays off well, starting right away.

## Sometimes things are non-goals

There’s two ideas that frequently get hailed as major design goals that I want to call out: **extensibility, and re-usability**.

These are referenced so frequently as an unmitigated good that I think we need a counter-reaction movement against them. It is absolutely not the goal of just any good code to be extensible and re-usable.

The reason these aren’t just the best things ever is that they’re both design goals related to [system boundaries](https://www.tedinski.com/2018/02/06/system-boundaries.html). A system boundary is any code that we can’t easily change the design of, usually because we don’t easily control all users of the API. In other words, where we suffer from the possibility of breaking changes and compatibility problems.

Extensibility and re-usability are potential goals **for system boundaries only**. To make a design extensible or re-usable is to make it more like a system boundary. You’re creating a lot more users that (especially for extensible designs) have much deeper coupling to that design, and it makes it a lot harder to *change* that design in the future.

System boundaries are the danger zones of design. To whatever extent possible, we don’t want to create them unnecessarily. Any mistakes we make there are frozen, under threat of expensive breaking changes to fix.

Instead, much of the time we want a design that’s **easy to evolve**, and the number one thing that makes something easier to change is **not** making it a system boundary. Most of the problems we discovered [back when we looked at the problems with inheritance in OO languages](https://www.tedinski.com/2018/02/13/inheritance-modularity.html) were exactly about this: extensibility creating design dependencies so subtle, it seems we can’t change anything at all without potentially breaking something (i.e. the “fragile base class” problem).

We may actually want to build an extensible system boundary, but we should be sure the design *should* be a system boundary, first. It’s remarkably easy to think “yeah, this should be extensible” and only later discover that decision has frozen almost every aspect of the software’s design, and now nothing can be fixed without potentially causing a lot of pain at the same time.

Likewise, re-usability seems weirdly fetishistic among programmers. It’s seemed to be an explicit goal for a huge part of the history of programming. But again, creating something that’s re-usable is pretty inherently about creating something that’s a system boundary, to some degree or another. And if you’re knowingly working on a system boundary… you’re knowingly working on something that supposed to be re-usable already. It’s pretty redundant to hail it as a design goal.

What’s often desired (instead of re-usability) for code that should not be part of a system boundary is instead **replaceability**. We want the easy ability to *delete* the darned code when we don’t want it around anymore. Instead of trying to more easily create dependencies on it, we want to better isolate it, so there’s less dependencies forcing us to keep it around.

This isn’t really a novel idea, but I don’t think I see it articulated as a design goal enough. Encapsulation is supposed to hide internal implementation details, but we often think about it in terms of hiding representation. Another consequence of hiding internal details is that those hidden pieces of code are easily discarded as things change. If a private method is obsolete, you can delete it; you don’t have to worry about someone, somewhere out there who might still need it, and so be forced to keep it around for backwards compatibility. So part of what encapsulation does is actually already encouraging replaceability in design.

The best designed code is code that doesn’t exist.

## Good design is all about constraints

Besides solving a problem *quickly*, the only other thing good design has to do for us is actually solve the problem. And a solution to the problem has to meet that problem’s constraints. These can be about performance, usability, portability or compatibility, scalability, fault-tolerance, and so on. I have less to say about this, because a lot of this stuff is going to be domain-specific. The design of a compiler and a highly scalable web service back-end is quick obviously going to be very different. This part isn’t really the focus on this series.

But I feel compelled to at least point this out. Speed isn’t actually *everything*, just most of what we’re concerned with if we’re talking about design in general. Some concerns are shared by almost all programming.

## So how do we do it?

Well shucks, that’s what I’m trying to answer with [this whole book](https://www.tedinski.com/book/). There are certainly things we can learn that make us better at doing design. Some of these are rooted in mathematics, like understanding [the expression problem](https://www.tedinski.com/2018/02/27/the-expression-problem.html) or [variance](https://www.tedinski.com/2018/06/26/variance.html). Some are rooted in human psychological biases we need to resist, like our [preference for more power](https://www.tedinski.com/2018/01/30/the-one-ring-problem-abstraction-and-power.html). Some are rooted in working against (theoretically fixable) past design mistakes the industry forces upon us, [like an inability to appropriate choose between objects and data in type design in current languages](https://www.tedinski.com/2018/01/23/data-objects-and-being-railroaded-into-misdesign.html).

#### Comments

[Comments section for Patreon backers](https://www.patreon.com/posts/20609033).

[Patreon](https://www.patreon.com/tedinski)

This post is part of a [book writing effort](https://www.tedinski.com/book/). Consider [supporting me on Patreon](https://www.patreon.com/tedinski).

#### Read more

- Follow me on Twitter: [@tedinski](https://www.twitter.com/tedinski/)
- Follow my [RSS Feed](https://www.tedinski.com/feed.xml)
- Check out the blog [archive](https://www.tedinski.com/archive/)
- Prev: [Why an interface with only one implementation?](https://www.tedinski.com/2018/07/31/interfaces-cutting-dependencies.html)
- Next: [Breaking systems into modules](https://www.tedinski.com/2018/08/14/modularity.html)

Copyright © 2018 [Ted Kaminski](https://www.tedinski.com/about/).