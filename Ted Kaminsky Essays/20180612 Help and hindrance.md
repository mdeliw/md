# Whereas types can be pervasive

Jun 12, 2018

*(Some people are annoyed by my attempt to slide into the tests/types subject carefully, if you’re really just trying to read this post as fast as possible to see if I have anything new to say, you can skip down to the first heading below.)*

I’m always a little hesitant to wade into the dangerous territory of arguments people get heated about. The worst is when I don’t really have a good sense of what’s going to get me in trouble. There’s the stuff people *say* they’re arguing about, but then why is everyone angry, because (in this case) that’s usually just a discussion about some tools? How could people get so upset about that?

Well, that’s a stupid question. After all, we’ve all gotten upset about this stuff at some point, right? But then, maybe we don’t always really understand why we do the things we do.

I’m a little more happy to wade into it when I have a handle on what’s actually gone horribly wrong. I am a bit [more willing to criticize object-orientation](https://www.tedinski.com/2018/02/13/inheritance-modularity.html) (well, really inheritance) because I think I understand what’s happened there. There’s a very specific poor design at the core of the whole, well, not concept in the abstract, but ideology as practiced. Moreover, you can try to deconstruct what “OO” means, and reassure its advocates that most of the pieces are perfectly fine. (Hence, [my attempt to describe an OO language without inheritance](https://www.tedinski.com/2018/02/20/an-oo-language-without-inheritance.html).)

But I know that’s not what gets people riled up exactly. It gives me confidence to chime in, because I feel like I have a fact I truly understand and have complete confidence in, but facts aren’t what really gets people upset. I personally value inconvenient facts, and so I’m sometimes happy to stand on my little rock and demand the storm part around me. But I’ve been on the sidelines watching too many arguments to believe that this will truly change minds, even for perfectly rational people. Usually the tactic is justified as helping onlookers make up their minds, no more.

The heat usually comes from something that’s being left unspoken in those arguments, and this industry is loaded with baggage hanging unspoken on technical arguments. Our whole society is, really, just look at our awful politics. But this industry is specifically plagued with a weird superiority complex thing. I distinctly recall (despite the intervening 14 years) my first year as an undergraduate student, watching two of my peers have a pissing match before class about who knew more about programming. (And smugly laughing quietly to myself when one came back with “well, I know XML.” I mean, I’m not immune to this shit, either.)

This industry is loaded with people suffering from imposter syndrome, and unsurprisingly also with jerks trying to police who’s a “real programmer.” Just look at the toxic interviewing culture and the strange hoops we make each other dance through to *prove* themselves. Is it really any wonder that criticism of a tool someone likes could be felt as criticism of them, personally? I certainly remember late-90s criticism of Java because “real men” could handle pointers in C++. That even manages to be toxic on multiple levels. If we asked ten people why they felt attacked by criticism of a tool, we’d probably get ten different answers, and they probably wouldn’t be wrong. Not exactly, anyway. After all, some jerk out there probably doesn’t care much about the technical arguments, only about how those arguments can be used as excuses so they can makes themselves feel superior.

I haven’t the slightest idea what to do about any of this.

So I’ll just stick to my first tactic. Today I’d like to talk about why I think static types aid design, by picking a small fact I think is important and standing on that little rock.

## Help and hindrance

One of the things Kent Beck (if you don’t recognize the name, just roll with “the TDD guy”) likes to point out is that writing tests relieves anxiety about coding. Without some sort of feedback, he worries too much that he’s missing something. This was part of his own motivation for writing tests first: it gave him some understanding on whether what he was doing was at all on track. Passing tests confirm it works as you expected, and writing failing tests for new features confirms it’s not yet working, as expected.

Of course, we get a very similar mythos from typing advocates. Advocates of languages like Haskell almost all have some fantastic story about writing some bit of code, wrestling with type errors for a bit, *and then it just worked!* It’s an exaggeration to say that “if it type checks it works,” but you’re going to end up with some experience that makes such a thing *seem plausible* if you try it long enough. My own experience of this phenomenon took the form of refactoring: taking a working program, changing a type around in a big way, causing hundreds of type errors, chasing them all down, and then… wow. That’s it? I didn’t manage to forget anything? It… it all just works again? It really seems amazing.

And refactoring plays into testing too. In fact, many people seem to consider “refactoring” to specifically refer *only* to the practice of making a change in the presence of a sufficiently good test suite to check whether behavior has stayed the same. Without that test suite, they don’t consider it “refactoring.” (I suspect, but am not completely sure, that this might come from Martin Fowler’s book?) I think this is slightly too narrow—I think there’s plenty of refactorings that require you to change the test suite, so it’s not a perfect measure of whether you’ve succeeded—but that’s not too important. Clearly, for many changes, the test suite is an aid.

Meanwhile, static types are routinely responsible for making automated refactoring possible. Or at least, making it possible to rely on automated refactoring. Certain tasks, like renaming a method, are only always doable if we’re able to find all references to that specific method (and only the references to that method), and we’re just not able to do that well enough to be especially useful without static types. (Of course, static types alone aren’t enough, as users of C++ will attest.)

So it’s probably no surprise that some people argue about “types vs tests.” They sometimes seem to be about the similar things, with similar effects. There’s no “versus” about it, really, but to get there, let’s find a rock to stand on.

## One thing that makes them very different

I’m not starting from the beginning with this one. I assume you have some context on this debate. Sometimes people like to talk about “there exists” contexts and “for all” contexts. But I digress.

One big difference is what happens as we write and maintain our programs over time. [Testing needs to be concentrated on harder system boundaries](https://www.tedinski.com/2018/04/10/making-tests-a-positive-influence-on-design.html). If you write tests on non-system boundaries, you’re creating a mild form of technical debt. What if you want to change that design later? The tests become an obstacle that slows you down; they have to be changed, too. Hard system boundaries are already hard to change, so those are safe to heaps test upon.

I don’t think this observation is entirely new. There’s certainly ideas out there about “test damage” to design. There’s certainly known ways in which tests can make it *worse* to work with a piece of code, rather than better. And people are sometimes aware that “brittle” tests can make it harder to change code. Framing things in terms of system boundaries is partly a way to understand *when and why* these things can happen. A test can’t make your code harder to change if it’s written against an interface that’s already hard to change.

But the big idea to today’s post is simply this: **this isn’t true of types.** There simply isn’t a place where, if you put types, it makes your code that much harder to change. Using a statically typed language can make code harder to write in the first place… maybe. (I still feel that most cases where this happens it’s really the language’s poor design at fault, and we could design better languages, but then I’m a programming languages PhD, I would say that, wouldn’t I?) But then over time the existence of those types generally makes everything easier. And it’s not just the automated refactorings, it’s the feedback the compiler is able to give when you manually make invasive changes to those types.

My mind reaches for a metaphor like concrete and steel. Different building materials, with different properties. You wouldn’t want to build a foundation out of steel. It’s not going to end well.

I thought that I would make a joke here about how no one would try to argue about whether concrete or steel was *best* building material. They’re obviously complementary right?

But I decided to google that before publishing and… huh. Okay, so I guess maybe we’re not so different after all.

So we have two different techniques for trying to make our programs work correctly, but one is best suited to system boundaries, and comes with drawbacks when applied elsewhere. The other is happily pervasive throughout the program, but we had to adopt it wholesale up-front. (I’m still “we’ll see” about gradual typing. Let’s hope that works out.) With modern type inference, you may only ever need to write out types explicitly on system boundaries, and the rest of the program can be inferred safely.

I think this little observation marks a big difference in how these two approaches affect our programs. We need static types after a certain point, because without them you’re just finding creative ways to build skyscraper without any steel. There’s just an easier way. The design decisions you make under those constraints are just going to leave people wondering why you put yourself into those constraints.

## End notes

- (Added 2018-8-7) I reworded some things. Some readers objected to my characterization that types don’t slow things down, or accused me of advocating for all the possible advantages of types… which don’t actually exist in combination in any real language, and could conceivably even be in conflict. That’s true, and I sort of already admitted it with my joke about being a PL PhD. For what it’s worth, I think my argument here still holds up if we just consider modern Java. Yes, there are plenty of ways in which writing a modern Java program can be slower than writing (say) a Python program. But increasingly, I think people are starting to come around to the realization that maintaining old dynamically typed software can be quite awful, and that Java with a few of its worst rough edges shaved off (new features in 7 and 8, abandoning the worst Enterprise frameworks) can… actually be surprisingly nice. Very contrary to its old reputation. But of course now I’m just asserting this without evidence. Oh well.
- It is irksome that gradual typing is so hard. It’d be great if we could choose to do something quick and dirty *and then fix it eventually* when it starts to obviously matter. But right now we’re all too often faced with complete rewrites instead. I think this contributes to the ferocity of the debate. Some people just really don’t want to have to *completely give up* dynamic languages. And it is sort of ridiculous that they’d have to.

#### Comments

[Comments section for Patreon backers](https://www.patreon.com/posts/19421496).

[Patreon](https://www.patreon.com/tedinski)

This post is part of a [book writing effort](https://www.tedinski.com/book/). Consider [supporting me on Patreon](https://www.patreon.com/tedinski).

#### Read more

- Follow me on Twitter: [@tedinski](https://www.twitter.com/tedinski/)
- Follow my [RSS Feed](https://www.tedinski.com/feed.xml)
- Check out the blog [archive](https://www.tedinski.com/archive/)
- Prev: [Small is not the same as simple](https://www.tedinski.com/2018/06/05/small-is-not-simple.html)
- Next: [All advice comes from a context](https://www.tedinski.com/2018/06/19/all-advice-has-context.html)

#### External Links

I noticed these discussions of this post: [Reddit](https://www.reddit.com/r/programming/comments/8qyioh/types_are_pervasive/)

Copyright © 2018 [Ted Kaminski](https://www.tedinski.com/about/).