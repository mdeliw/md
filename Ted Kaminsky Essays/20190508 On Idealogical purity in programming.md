Ted Kaminsky Essays

[toc]

# On Idealogical purity in programming

May 8, 2019

One of the things that plagues this industry is the promotion of good rules of thumb into absolute rules. What I’d like to do today is just muse a little bit on how this happens, and what we should do about it.

## Moral reasoning

I’m no psychologist, so take this with a grain of salt. But it seems to me there’s some part of our brains that does moral reasoning by a mechanism that’s different from how we do “rational” reasoning. In part, this is not a dysfunction, it’s an optimization. To do moral reasoning in a “rational” way, you have to get really systematic about expanding out all the downstream consequences, intended or not, and even foreseen or not. Which is… rather difficult to do.

Moral reasoning is a shortcut that says “Nope! Stop. That’s just wrong; don’t go that way.” Don’t waste time trying to reason out the impossible infinite unknowable branching decision tree here, because we have a moral principle we trust that says “this will not end well.” 

But it’s not *just* that, because this is the basic idea behind “thinking fast” and “thinking slow.” Or in other words, intuition. Intuition is fine; we’re already reasonably capable of recognizing when our intuitions have lead us astray. More reliable intuition is part of what experience is supposed to give us.

Moral reasoning seems to be accompanied by some extra “unquestionability.” We’re all able to follow our intuitions and then stop and think “ehh, maybe this isn’t working.” But following “moral” principles can take us in a direction where we suffer the consequences and insist on sticking it out instead.

There’s even some kind of vague expectation that we’ll eventually be rewarded for sticking to the principles, even if there’s no reason to believe that’s actually true.

## Ideology

I’m no… I don’t even know what, but take even more salt here. What distinguishes an *ideology* from just any old set of beliefs, I think, is that an ideology specifies moral beliefs.

To use a nice uncontroversial example not directly related to programming, there’s “capitalism” and then there’s “Capitalism.” First, we have a neutral mechanism for organizing an economy. A purely mechanical thing about which we can study and make observations. An intellectual curiosity.

The second tacks on *moral* judgments like “the outcomes of a free market are just” or “the shareholders *deserve* the profits.” Understandably, when this ideology asks us to conclude “teachers and nurses *should* be paid poor wages,” people start wondering if maybe we should do away with that whole (at least big-C) Capitalism thing.

## Technical ideologies

I bring all this up on a blog about software design because we suffer from this all the time in this industry. The thing is, it’s ideologies about *programming*. Ideologies de-facto introduce the idea that there’s a *“morally righteous”* way to write a program, as odd as that sounds. I hope that sounds odd to you.

Object-orientation’s splash onto the scene came with a lot of associated ideology. One of these was that `switch` was to always be avoided in favor of polymorphism and dynamic dispatch. But this is just a blanket ban on [data types](https://www.tedinski.com/2018/01/23/data-objects-and-being-railroaded-into-misdesign.html) which just cuts off a significant portion of the [type design space](https://www.tedinski.com/2018/02/27/the-expression-problem.html) for no real reason.

I’ve written articles about the [flaws of inheritance](https://www.tedinski.com/2018/02/13/inheritance-modularity.html), but (in a way) even this proscription takes on ideological overtones. I do think we’d be better off designing programming languages without the feature, but as long as it’s here, we can also look at [when it’s pretty safe to use it](https://www.tedinski.com/2019/04/16/composition-vs-extension.html).

“Goto considered harmful” has entered into the history of programming in such a big way, I don’t even feel the need to link to something as a reference. But people sometimes get surprised when it’s used regularly in [the Linux kernel](https://lwn.net/Kernel/LDD2/ch02.lwn#t4) and other C software, and for pretty good reason. They’re even more surprised to discover that virtually every complaint about `goto` that Djikstra made actually doesn’t apply all that well to `goto` as it exists in C. (Not that this makes `goto` good or anything, but the reasons *why* have to change!)

So much in testing has been turned into ideology. TDD is [sometimes](https://www.tedinski.com/2019/03/19/testing-at-the-boundaries.html) an effective practice, but it frequently get turned into a moral purity test. Devotion to high coverage and unit-only (lest ye sin with integration testing?) tests can lead to “[test-induced design damage](https://dhh.dk/2014/tdd-is-dead-long-live-testing.html)”. My observation that [tests against non-system boundaries can be a kind of technical debt](https://www.tedinski.com/2018/06/12/types-can-be-pervasive.html) still seems rather transgressive.

I think I’ve told this story before on this blog, but I’ve had to delete a unit test suite before. It took me a long time to come around to accepting this because I kept thinking that this unit test suite was *good*, and that deleting it would obviously be *bad*. But… it kept causing problems, and never really provided any protection against introducing new bugs. It was another long time after I had gotten rid of it before I had any sort of “principled” justification for why that was the correct decision beyond, “look, it just breaks all the time for no good reason.”

And the best ideologies are the ones we don’t even know we believe. I routinely see automatic, unquestioned praise for “extensible designs,” but I happen to think [extensibility is a rather significant cost, to be paid only where necessary](https://www.tedinski.com/2018/08/07/what-makes-good-design.html). It’s not automatically a feature of good design.

## Why do we do this?

To some degree, I think it’s inevitable. We’re trying to persuade humans to follow some rules in the short term in order to try to reap some longer-term reward we might not immediately see or appreciate. That’s… a kind of persuasion that’s always going to touch a little on the “moral” side.

Partly, I think this kind of thing is desirable. Like I said, this overlaps a lot with the idea of “thinking fast” versus “slow.” We do actually need to build intuitions, and we need to be able to apply those intuitions. The trouble is just that we so easily slip from “good rules of thumb” to “you’re bad for even questioning this.”

But another reason I think this happens to us is that these ideologies aren’t really invented in one go. They evolve. An ideology has to propagate somehow, and thinking about this in terms of evolutionary fitness is a bit enlightening. One way to propagate pretty well in this industry (unfortunately) is to give people reasons to feel morally superior to those around them. Adoption of the ideology becomes a “benefit” for more than just engineering reasons.

Of course, that’s also completely toxic. Our especially strong susceptibility to this sort of “pecking order” crap is a bit of an indictment on programming culture.

## What should we do about it?

The biggest problem here is that we’ve often arrived at ideological beliefs without really having *reasoned* our way into that position. Ideologies are frequently taught unconsciously. Either by mimicing the value judgments we see others make, or simply by explicitly being told to look at things a certain way before we’re really able to critically evaluate it. Doing “this” is bad, and you should do “that” instead. Okay. This thing is bad, and if I do it, I’m bad, right?

Experience and intelligence don’t necessarily save us from this, because it’s not fully conscious. Experience and intelligence can perfectly well *just* help us construct ever more sophisticated rationalizations for the ideologies we haven’t questioned. It’s not until we start to allow experience to point out the flaws in those ideologies that we can start to become wiser.

But pointing out the flaws in ideologies is fraught. We have a moral reflex: if someone questions a moral principle, that’s a sign of their *immorality*, right? Maybe that person questions TDD because they’re a bad programmer?

I can’t fix this for you, but I can make a suggestion. If you find yourself about to say “no, that’s a bad and wrong approach” to a technical decision, instead apply the Socratic method. Ask “why would we want to go with that approach?” Ask if the approach has any obvious drawbacks, or obvious advantages. Think, ask, explore, look.

For one thing, this often a more effective method for teaching a junior engineer. If you’re actually quite right, you should be able to get them to see that by asking the appropriate questions. You don’t need to hand down laws. And who knows, maybe they’ll surprise you with a good answer.

## End note

- As an aside, I’m not sure I really believe there are “idealists” or “pragmatists”, or that these are helpful ways of thinking about people. One could misconstrue what I’ve said above as endorsement of pure pragmatism, but I wouldn’t agree with that. The misuse of ideology in programming isn’t something an idealist would be interested in doing either. This is a separate behavior, common to all.
- I guess I’ve made something of an argument against “moral reasoning” about technical decisions, so I should clarify: ethics of course matter to developers! But… the ethics of software is about the impact we have on the world around us, not *just* (e.g.) the narrow minutia of specific testing methodology.