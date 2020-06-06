# Testing at the boundaries

Mar 19, 2019

I don’t normally have that much appreciation for “classics.” At the risk of outing myself as a heretic, Tolkien for example does great world-building and… that’s about it. He may have defined a genre, but I really think other authors have done a better job with, like, plot. And characters.

I say this because I think there’s a lot to learn about TDD from [Kent Beck’s original book](https://www.oreilly.com/library/view/test-driven-development/0321146530/). Last week, I was inspired to muse a bit about how [the TDD process “as described” is really about dynamic object-oriented programming](https://www.tedinski.com/2019/03/11/fast-feedback-from-tests.html), and more-typed “more-functional” languages interact differently. This week, I want to point out that a lot of the things I most disliked about TDD when I encountered it “in the wild” *do not appear in this book*.

## System boundaries, implicitly

I recently tweeted a [good talk by Jessica Kerr about property testing](https://www.youtube.com/watch?v=shngiiBfD80), and one of the interesting things is that later in the talk, she brings up [system boundaries](https://www.tedinski.com/2018/02/06/system-boundaries.html). Except… not by name, because we don’t seem to have a widely accepted name for this concept, which is part of what I’m trying to cure with this book project.

One of her off-hand remarks is that, if you think you have to write unit tests for every method of every class, you’ve been mis-educated. This was a point I started trying to make [last year](https://www.tedinski.com/2018/04/10/making-tests-a-positive-influence-on-design.html). This is a pretty concrete form of dysfunction: somehow, somewhere out there, a lot of people have started to get the idea that this is what TDD was about. And… it sorta makes sense, because to do TDD, you kinda have to *start* that way.

Kent Beck’s book opens by example, and right away in the *very first example* he displays the opposite behavior! The opening example is all about representing money in different currencies. He starts simple with `Dollar`, then starts to consider how to introduce `Franc`. This precipitates a `Money` interface, *and then a refactoring of all the tests to purge direct references to each concrete class implementing `Money`*. (For example, even `new Dollar(5)` becomes `Money.dollars(5)` in the tests, so that no references remain at all. Naturally, if these had started life in `DollarTest.java` they’d probably move, too.)

The purpose of this refactoring is to allow these class implementations to freely change, unconstrained by the tests. This is accomplished by moving the tests from the internal implementations to the system boundary (or at least, to the “harder” system boundary).

This isn’t directly examined or highlighted in this way, but the motivating factor here is that *writing tests against the concrete classes was technical debt*. That was where we had to start—the `Money` interface didn’t even exist at first—but as the design evolved, the old tests became a liability that needed to be paid down. The tests needed to be refactored before things proceeded.

I find it very curious that the book that started TDD begins with an example showing it’s a bad idea to insist on “unit tests for every method of every class,” and yet that’s where some of the industry went. I may have to reconsider my lack of appreciation for classics.

### How did we get here?

This is pure guesswork, but I think human brains are just a bit broken. For humans, ideological reasoning is moral reasoning. TDD isn’t just another technique, it’s *what’s right* and deviating from it is *wrong*.

And if there’s one thing the human brain likes, it’s when doing the right thing leads to the right outcome. So the idea that following TDD might *create technical debt* is repulsive. Now, TDD can’t be pure and just and good, because it doesn’t always deliver us from evil.

And this is a seductive idea, since following TDD leads us to have tests! Tests good! Right?

So we went from “drinking water is good for you” to “the only moral process for developing software is force-feeding our developers 150 L of water a day. Please ignore the bodies in our wake.” Oops.

## TDD falling down

So last week, I mentioned how “data + static types” can change how TDD works, and noted how things like DB schema design (type design) aren’t really as amenable to TDD as a process. Today, I’d also like to relate a story about ordinary coding where TDD falls down as well.

I’ve mentioned before that [compilers rarely unit test](https://www.tedinski.com/2019/03/19/2018-04-10-making-tests-a-positive-influence-on-design), but I’d like to expand on that idea with my own story.

Years ago, I redesigned the type system for a whole programming language. This was an academic language, and its compiler wasn’t in the best state when I started. Part of what I was also trying to accomplish was improving our testing, to ensure we didn’t accidentally break things for our users.

So when I wrote the new type system implementation, I wrote a test suite along with it. This wasn’t exactly TDD (I feel compelled to note, even after describing how TDD can change with types last week), but it was pretty close. The point was the same: write a little code, and quickly exercise it to get confidence it works, in the form of a test suite.

This went quite well! Once the machinery was working, I wired it up into the rest of the compiler. Tadaa! A replacement type system, complete with new unit test suite for it.

Years later, we deleted that entire test suite.

Now, I don’t regret writing it in the first place. It was quite helpful when I was initially developing that code. It’s a long road from writing a unification function to being able to finally run it once it’s used everywhere in the compiler implementation. Starting by writing a function and immediately unit testing it was great. It easily sped up every bit of the development process: writing the code (less time thinking “wait, is that right?”), debugging (“oh, *this* thing isn’t working yet!”), and so on.

But we didn’t end up with a useful test suite afterwards.

The problem was that every change after the type system was merged fit in one of two categories:

1. The change never broke a test, nor really could it have.
2. The change broke lots of tests, and it was a design change that *should* have broken lots of tests, because the tests became wrong.

So every change we made meant coming back to a pile of broken tests, and just… updating the “expected” values of each test. We just copy & pasted the new “actual” values from the test failure.

This is why we deleted the test suite. It never helped, and this was just a pointless hindrance.

The type system was better exercised with tests written against inputs to the compiler. “This broken code should raise a type error about this.” This style of test has two advantages:

1. Because it’s written against a system boundary (the language the compiler accepts), it can almost never become technical debt and impede progress.
2. It not only exercises the typing machinery, but also the connection between expressions, the local environment, and the typing machinery.

The downside, of course, is that these kinds of tests can’t be run right away while coding. Like I said, I didn’t regret unit testing that code *at the time*. It just didn’t leave me with a useful test suite afterwards.

## Unit tests are not all there is

The TDD approach is powerful in part because it comes with tools to help make things happen. You aren’t just instructed to write tests, you’re given a test running harness like JUnit to help you run them.

But another drawback of the TDD approach is that *this is limiting*, especially in your thinking. Many things just aren’t done well with example-based unit testing.

Here are a few examples.

### Games or simulations

How do you test a game? The game industry is notoriously terrible with testing. At a guess, there are three causes:

1. Terrible working conditions. Yikes.
2. Many “one and done” products. Many games are not maintained long-term after release.
3. Wait, how **do** you test a game?

The problem here is that unit testing is far less applicable. Simulations depend on lots of highly-interacting components, written in a high-performance style. It’s nice when you can find a thing here or there amenable to a unit test, but a lot of stuff is just not obviously testable!

Or at least, it seems not testable, when all you’ve got is a unit testing harness.

The secret is to build custom test harnesses. Maybe even game-specific ones. Multiple. This stuff isn’t actually untestable, it’s only that general-purpose xUnit-style testing fails us. (This isn’t my area of expertise, so I don’t have good examples to give you here, however.)

### Concurrency

Concurrent systems are one of the original “maybe not so applicable” areas for TDD, but it turns out, you **can** test them. Just… again not with the “up-front unit testing style.”

I’ve mentioned before, and will never get tired of talking about, how cool [Jepsen](https://jepsen.io/) is for doing deep property-testing of distributed databases. But it’s property testing, and it’s a non-unit testing style that doesn’t really give you design guidance as you go.

### Distributed systems and monitoring

One of the most popular, and remarkably effective, testing strategies today is called “testing in production.”

That’s hopefully slightly a joke, but simultaneously very true.

We monitor running systems for failures. We collect crash reports from user’s machines. We log exceptions that occur when our Javascript runs in our user’s browsers. We do “canary deployments” of new services, allowing us to discover any new failures quickly without impacting a large number of users.

Again, these are great testing strategies that don’t really fit into the TDD worldview.

### Iteration and feedback

Design is always iterative, and TDD is one way to do iterative design when it comes to coding an implementation. But that doesn’t mean the kinds of iteration TDD brings is the right kind of iteration. You might just be spinning your wheels.

When talking about distributed systems, I’ve been trying to stick to the “happy path,” where we offload all the complexity onto databases, and otherwise just operate some simple services. But sometimes your job is to write those databases. What should you do then?

One answer is something like Jepsen, but Jepsen is also not a very TDD-ish approach. It doesn’t give us design feedback as we’re writing the code, it just helps us find bugs in the implementation after the fact.

Another answer might be TLA+. [Hillel calls this design up front](https://twitter.com/Hillelogram/status/1107459272953262090), but that’s only true if you think “before you start implementing code” as “up front design.” But the processes of *writing the TLA+* is iterative, because design is always iterative, and we’re getting fast feedback from the machine while we do it. We’re just operating on a higher level of abstraction, one well-suited to the problem we’re trying to solve. (And we’re working with something where TDD isn’t really applicable as an approach anymore, too.)

In that tweet-thread, someone links to [an interesting pile of links](https://ravimohan.blogspot.com/2007/04/learning-from-sudoku-solvers.html) where people contrast TDD with “thinking.” I think it’s really unfortunate, and a definite dysfunction of TDD advocacy, that “thinking” became contrasted with TDD. TDD is a great way to write code, when you already know what code you generally need to write. But sometimes you have to think at a higher level of abstraction first. TDD shouldn’t be contrasted with thinking, TDD should be contrasted with “okay, now that we know what we’re going to do, let’s plan out all the UML diagrams before we starting coding…”

Always think ahead about *problems*. I’ve always felt like TDD is solely about bridging the gap between “I think I know how to solve this problem” and “this is what the code to solve this problem will actually look like.”

## End notes

One general thing that’s not lost on me is that the “bad TDD” approach in that previous link spins its wheels thinking about representation, instead of solving the problem. Kent Beck’s book also spends its time thinking on representation (of `Money`), too.

One might be tempted to accuse TDD of basically being an approach to coping with the deficiencies of dynamic object-oriented languages when it comes to representing data.

But I think that’s unfair. “Static types + data” gives us tools to reason about representation much better, but we still often need to course-correct our representations as we discover new things during the process of implementation.

What’s different is how we do that. When you realize your types need changing in the middle of implementing the next task, you can pause what you’re currently doing, change the types, fix all the new type errors, and return to the task at hand.

The process is a lot more involved without the aid of static types, or without rich enough data types (such as with Java). [This tweet from yesterday about TypeScript from Gary Bernhardt comes to mind.](https://twitter.com/garybernhardt/status/1107738508288876544)

#### Comments

[Comments section for Patreon backers](https://www.patreon.com/posts/25484757).

[Patreon](https://www.patreon.com/tedinski)

This post is part of a [book writing effort](https://www.tedinski.com/book/). Consider [supporting me on Patreon](https://www.patreon.com/tedinski).

#### Read more

- Follow me on Twitter: [@tedinski](https://www.twitter.com/tedinski/)
- Follow my [RSS Feed](https://www.tedinski.com/feed.xml)
- Check out the blog [archive](https://www.tedinski.com/archive/)
- Prev: [Quick feedback loops while coding](https://www.tedinski.com/2019/03/11/fast-feedback-from-tests.html)
- Next: [Deconstructing SOLID design principles](https://www.tedinski.com/2019/04/02/solid-critique.html)

Copyright © 2019 [Ted Kaminski](https://www.tedinski.com/about/).