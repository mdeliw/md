# How humans write programs

Jan 16, 2018

Since this is the inaugural post of this new project, I should mention that this blog is a tool to [help me write a book about software design](https://www.tedinski.com/book/). This is the beginning of that effort.

I’d like to get to technical content as quickly as possible, but programming is also a human activity, and so there are things I (for whatever reason) feel compelled to talk about first. Part of this is framing: what would you even get out of a book about software design? Would you get better at designing programs up-front? I hope so, at least a little. But an even bigger issue, I think, is being able to come up with designs that are actually better than the one you have already. This comes down to two things: imagination and judgment. You have to think of something better, and also be correct in your assessment that it is, in fact, better.

When we set out to write a program, what is it that we are doing? Most programming is maintenance or evolution of existing software, not greenfield development. But even so, I think the process is actually pretty similar.

*Definition:* **The standard algorithm**

1. Write stuff
2. Fix it
3. Don’t forget step 2

This approach gets a bad rap. It seems like it’s completely without intelligent planning. The aphorism goes “measure twice, cut once” but the above might make programming sound like “measure never, cut repeatedly until it works!” Surely we can’t all be unprincipled hacks?

But programming remains at least partially a creative process—there’s a design element to code. Just about any creative process requires creating something pretty bad, and then fixing it. Nobody creates something good on their first try, having made no mistakes. We don’t really understand the thing we’re trying to create—at least not until we’ve given it a go.

Hence, the standard algorithm. And our industry’s preference for “agile software development.” And our general derision towards the idea of rewriting software, and our preference for refactoring software. Programming comes with the added challenge that you’ll never fit all the details in your head. You have to somehow offload them in a form where they can be meticulously checked (such as running code).

Software isn’t really alone in this. From military planning we get the [OODA loop](https://en.wikipedia.org/wiki/OODA_loop). Creative processes of all kinds acknowledge the need to create a pile of shit, from which you can find a few gems. The most common blame for “writer’s block”—the inability to make any progress writing a book—is that the author believes they have to write something *good* and not merely write *something* and then later try to make it good. But it’s hard to exclusively create good things, and so they lock up not creating anything.

Excerpt: **Surely You’re Joking, Mr. Feynman!** (p. 173–174)

So I got this new attitude. Now that I *am* burned out and I’ll never accomplish anything, I’ve got this nice position at the university teaching classes which I rather enjoy, and just like I read the *Arabian Nights* for pleasure, I’m going to *play* with physics, whenever I want to, without worrying about any importance whatsoever.

Within a week I was in the cafeteria and some guy, fooling around, throws a plate in the air. As the plate went up in the air I saw it wobble, and I noticed the red medallion of Cornell on the plate going around. It was pretty obvious to me that the medallion went around faster than the wobbling. [Or vice versa.]

…“Hey, Hans! I noticed something interesting. Here the plate goes around so, and the reason it’s two to one is . . .”

“Feynman, that’s pretty interesting, but what’s the importance of it? Why are you doing it?”

“Hah! There’s no importance whatsoever. I’m just doing it for the fun of it.”

…There was no importance to what I was doing, but ultimately there was. The diagrams and the whole business that I got the Nobel Prize for came from that piddling around with the wobbling plate.

Software development consists of a long series of complicated decisions. Decisions that have to be made along the way, not just because it’s hopeless to decide up front, but because it’s hopeless to even know what they all are up front. Decisions that will often enough come down to whoever’s on the spot at the time, with vague information about what’s really at stake. Trying to get it all right ahead of time is like the worst committee meeting from hell, except it can happen all inside your own head.

![Phase 1: Code and apply experience. Phase 2: Evaluate and learn from experience. Repeat.](https://www.tedinski.com/assets/two-phase-model.svg)

I think it’s helpful to imagine a two-phase approach to software development. This is echoed in some of our practices: for example, sprints in scrum ask us to work a couple weeks, then pause to review and plan for the next sprint. (There’s other problems that development *process* has to confront, like communication within and between teams, and it’s not my intention to get into that.) We have to take the time to get into the details of a problem to solve it, and then we also have to re-orient our perspective away from the task at hand to think about the bigger picture. The “standard algorithm” might look like three steps at first, but it can take iterating awhile to really follow it.

So maybe we shouldn’t feel too bad about jumping in head first. But is there any hope for doing better? Maybe. I think the standard algorithm is so fundamental to the human creative process that we’ll never escape it. ([A video interview with Dan Harmon on the subject.](https://www.youtube.com/watch?v=RgT1OobFArg)) But it’s still possible to do some things in more principled ways.

One of the most basic tools in engineering design is to **create prototypes**. In software development, we’re sometimes scared of prototyping, because we’re plagued by managers who will think the prototype is almost a finished product. Prototyping is still a good tool, however, we just need to need to work around such obstacles. Prototypes allow us to experiment with designs, with enough detail to critique them, while investing significantly less effort into their creation than the finished product would require.

Another way we can do design in a more principled way is to **work at a higher level of abstraction**. This doesn’t change the standard algorithm, but it usually means working on a *smaller* solution, which goes faster. (And as a general rule of thumb, I think the time it takes to complete something is super-linear with its complexity, so perhaps I should say *much* faster.)

At the limits of abstraction, we can do formal verification. Writing *specifications* remains much like writing programs in that you have to be specific about all the important details, and all your assumptions are getting machine checked. (Most frequently today, machines check our assumptions when we run our code, and we discover it manages to not crash and burn. With verification, the machine can do much better.) And as a bonus, some specification languages let you animate specs into prototypes. And having prototypes lets you find bugs in the specification. Verification remains expensive and difficult, but I think it’s very much reached practicality for some things, in industries that find the expense worthwhile.

Lastly, we can do a better job of **focusing on where design matters the most**. Our programs generally have some core abstractions around which everything is built. Compilers have an AST. CRUD apps have a database schema. Much of the praise of git is based on the core design of how repositories are represented. (The praise certainly isn’t for its user interface.) Get right the core design problem, and let the rest of the application just hang off it. This can be a difficult thing to do, unless you actually take time to think specifically about it. Consider how many web apps have been written by people with strong opinions about [MVC](https://en.wikipedia.org/wiki/Model–view–controller)… using a schemaless database.

Thanks for reading. I’ll have an essay like this one every Tuesday.

#### Comments

[Comments section for Patreon backers](https://www.patreon.com/posts/16437052).

[Patreon](https://www.patreon.com/tedinski)

This post is part of a [book writing effort](https://www.tedinski.com/book/). Consider [supporting me on Patreon](https://www.patreon.com/tedinski).

#### Read more

- Follow me on Twitter: [@tedinski](https://www.twitter.com/tedinski/)
- Follow my [RSS Feed](https://www.tedinski.com/feed.xml)
- Check out the blog [archive](https://www.tedinski.com/archive/)
- Next: [Data, objects, and how we're railroaded into poor design](https://www.tedinski.com/2018/01/23/data-objects-and-being-railroaded-into-misdesign.html)

#### External Links

I noticed these discussions of this post: [Reddit](https://www.reddit.com/r/programming/comments/7r1uvd/how_humans_write_programs/)

Copyright © 2018 [Ted Kaminski](https://www.tedinski.com/about/).