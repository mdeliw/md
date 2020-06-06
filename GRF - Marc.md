GRF - Marc



You have more places that can time out, require new ways to detect failures and communicate them back to users, and so on.



Those can be worked around, don't get me wrong. The issue is that they're being introduced as part of a solution that's not appropriate for the problem it's built to solve. All of this was just premature optimization. Even when everyone involved took measures, reacted to real failures in real pain points, etc. The issue is that nobody considered what the true, central business end of things is, and what its limits are. People considered these limits locally in each sub-component, more or less, and not always.



Depending on where you put your probe, you can optimize for different levels of latency and throughput, but what you're going to do is define proper operational limits of your system.



Proper metrics of your quality of service

Fewer critical rushes to get everything fixed because it's dying all the time

A way to monetize your services through varying account limits and priority lanes

You act as a more reliable endpoint for everyone who depends on you



It's because of bad system engineering, where people are trying to make an 18-wheeler go through a straw and wondering why the hell things go bad. 



Asynchronous sends, such as those performed by Kafka, are not in themselves a workaround for back-pressure. An asynchronous send will involve messages being placed into a finite in-memory buffer that is periodically sent by a background thread to the broker. In the case of Kafka’s client library, if this buffer becomes full before the client is able to reconnect, then any subsequent sends will be blocked (i.e., become synchronous). At this time the application thread will wait for some period until eventually the operation is abandoned and a `TimeoutException` is thrown. At best, asynchronous sends will delay the exhaustion of an application’s thread pool.



**Make sure that failures are handled consistently**

**Make sure that failures are handled safely**.

**Make sure that failures are handled responsibly**.





**The standard algorithm**

1. Write stuff
2. Fix it
3. Don’t forget step 2

This approach gets a bad rap. It seems like it’s completely without intelligent planning. The aphorism goes “measure twice, cut once” but the above might make programming sound like “measure never, cut repeatedly until it works!” Surely we can’t all be unprincipled hacks?

But programming remains at least partially a creative process—there’s a design element to code. Just about any creative process requires creating something pretty bad, and then fixing it. Nobody creates something good on their first try, having made no mistakes. We don’t really understand the thing we’re trying to create—at least not until we’ve given it a go.

Hence, the standard algorithm. And our industry’s preference for “agile software development.” And our general derision towards the idea of rewriting software, and our preference for refactoring software. Programming comes with the added challenge that you’ll never fit all the details in your head. You have to somehow offload them in a form where they can be meticulously checked (such as running code).



One of the most basic tools in engineering design is to **create prototypes**. In software development, we’re sometimes scared of prototyping, because we’re plagued by managers who will think the prototype is almost a finished product. Prototyping is still a good tool, however, we just need to need to work around such obstacles. Prototypes allow us to experiment with designs, with enough detail to critique them, while investing significantly less effort into their creation than the finished product would require.



Another way we can do design in a more principled way is to **work at a higher level of abstraction**. This doesn’t change the standard algorithm, but it usually means working on a *smaller* solution, which goes faster. (And as a general rule of thumb, I think the time it takes to complete something is super-linear with its complexity, so perhaps I should say *much* faster.)



At the limits of abstraction, we can do formal verification. Writing *specifications* remains much like writing programs in that you have to be specific about all the important details, and all your assumptions are getting machine checked. 



Lastly, we can do a better job of **focusing on where design matters the most**.  



Get right the core design problem, and let the rest of the application just hang off it. This can be a difficult thing to do, unless you actually take time to think specifically about it.



