Modern Architecture Articles

# General

In [computer science](https://en.wikipedia.org/wiki/Computer_science), the **thundering herd problem** occurs when a large number of processes or threads waiting for an event are awoken when that event occurs, but only one process is able to handle the event. After all the processes wake up, they will start to handle the event, but only one will win. All processes will compete for resources, possibly freezing the computer, until the herd is calmed down again.



In [computer science](https://en.wikipedia.org/wiki/Computer_science), the **sleeping barber problem** is a classic [inter-process communication](https://en.wikipedia.org/wiki/Inter-process_communication) and [synchronization](https://en.wikipedia.org/wiki/Synchronization) problem between multiple [operating system](https://en.wikipedia.org/wiki/Operating_system) [processes](https://en.wikipedia.org/wiki/Process_(computing)). The problem is analogous to that of keeping a barber working when there are customers, resting when there are none, and doing so in an orderly manner.

The Sleeping Barber Problem is often attributed to [Edsger Dijkstra](https://en.wikipedia.org/wiki/Edsger_Dijkstra) (1965), one of the pioneers in computer science.



A **cache stampede** is a type of [cascading failure](https://en.wikipedia.org/wiki/Cascading_failure) that can occur when massively [parallel computing](https://en.wikipedia.org/wiki/Parallel_computing) systems with [caching](https://en.wikipedia.org/wiki/Caching_(computing)) mechanisms come under very high load. This behaviour is sometimes also called **dog-piling**. At very high-loads, the cache expires and every request experiencing the cache miss will try to refresh cache. At high concurrency, no request is able to complete its task. 

# Back Pressure

> Resistance or force opposing the desired flow of data through software

## Strategies

- Scale up the Consumer
- Control the Producer - slow down or speed up is decided by the consumer
- Buffer - Accumulate incoming data spikes temporarily. Make sure its not un-bounded.
- Drop - Small percentage of incoming data is dropped 
  - Incoming data punched with a cost of doing manual process helps decide what to drop?

- Pull - Consumer controls the Producer

- Push - Producer is in control

- Repeatable process strategies

  - Throttle - The function is called at most once in a specified time period. It will prevent a function from running if it has run “recently”.

  - Debounce -  Function will ignore all calls to it until the calls have stopped for a specified time period. 

    

    > ==Here’s an analogy using a real-world scenario:==
    >
    > Suppose you’re working at your PC and also chatting with your friend Bill on a chat app (such as Telegram), who's telling you a story in bits and pieces (heh). You get push notifications every minute or so. Normally, you’d read every message as it comes in. But you're busy, so you can’t afford to switch context that often. What can you do?

    - Ignore your notifications, and decide that you’ll check your messages only once every five minutes.
    - Ignore your notifications. When no new notifications have come in for the last five minutes, assume Bill is done with his story and then check your messages.
    - The first approach is an example of throttling, while the second is an example of debouncing. Throttling is most useful when the input to the function call doesn’t matter, or is the same each time (for instance, a scroll event), whereas debouncing fits best when the result of the most recent event occurrence (for instance, when resizing a window) is what matters to the end user. 

## Taking the load off an overloaded system

https://www.tedinski.com/2019/03/05/backpressure.html

There are two basic strategies for taking the load off of an overloaded system.

- Exponential Backoff
- Circuit Breakers



Exponential Backoff is for closed-systems - where an overloaded system is receiving requests from a very small set of upstream systems, and the load pattern is predictable.  Backoff will be N, N/2, N/4 where N is the number of incoming requests.  At some point the load will drop to a level where requests are successfully executed, and the overloaded system recovers.



Circuit Breaker is for open-systems - where an overloaded system is received requests from a high number of upstream systems, and the load pattern is unpredictable.  In such scenarios, getting upstream systems to slow down doesn’t help when there are many upstream systems.  The fundamental problem is that the overloaded system just doesn’t have much information about the upstream clients, and hence needs a central process to “know things” - this is convered into knowledge and policy as part of the circuit breaker.

> If the incoming request’s error rate grows high enough (knowledge), then the circuit breaker trips the incoming requests and cuts of the load. 



The key thing a circuit-breaker is doing here is correlating multiple requests together. If the last thousand requests timed out, there’s no reason for the overloaded system to expect differently. The upstream system  just doesn’t know, and would be eagerly starting a request exactly as if the system were functioning. The circuit-breaker protects part of a distributed system from load coming from far too many sources.



Circuit-breaker has three states:

- Closed - requests are processed as normal.
- Open - error status around access to a resource is detected, subsequent requests are routed to an alternative logic.
- Half-open - Triggered after a period of time to allow requests to proceed to retest the protected reource. From here, the circuit-breaker will either be closed, allowing application logic to continue as normal, or re-opened.



Both Exponential Backoff and Circuit Breaker rely on the same thing - that is to have access to information to help reduce the load on the overall ecosystem, however: the upstream client (retrying node), and the overloaded system (circuit-breaking node) require information about the state the rest of the system to eventually slow down and later recover. To get that information, we need backpressure signals. Simply put, the request to apply back pressure needs to come from the overloaded system to the upstream system. These signals unfortunately are also going through the same communication pipes as regular requests that are facing overload scenarios and getting blocked, dropped etc - that is through the retry nodes and the circuit-breaker nodes. This means that the flow of the back pressure signal can be blocked, dropped etc, not allowing the information to slow down to be distributed across the systems. 



What we need to get to is that as long as system propagates backpressure, the load-reduction strategies can access the information to help slow down.  Backpressure is a *compositional* strategy - meaning we can build the whole from its parts. Any node of a larger system that correctly handles and exerts backpressure can be put together with other nodes that do likewise, and you end up with a whole system that handles and exerts backpressure. On the flip-side, if any node incorrectly handles the backpressure (such as a unbounded queue causing memory exhaustion), the “whole” can fall apart.

https://ferd.ca/queues-don-t-fix-overload.html



## Idempotency

The easiest way to address inconsistencies in distributed state caused by failures is to implement server endpoints so that they’re *idempotent*, which means that they can be called any number of times while guaranteeing that side effects only occur once.

A consumer managing idempotency can apply the following strategies:

- Each incoming request needs a unique key. 
- Consumer needs to remember which requests have been processed - lets say in a idempotent repository.  This is not easy to manage in the distributed world - its not an atomic instruction, and its also a race condition.  The following issues need to be handled:
  - Check if incoming request is already processed or not - a race condition. This includes reads and writes (add key to repo from below example)
  - If incoming request is not processed:
    - Add key to repo and then attempt to process the request. If the request succeeds all good, if the request fails, we need to remove the key from the repo.
    - Add key to repo after the request succeeds, this increases the risk of duplicate processing. 
    - System crashes before we can add the key to the repo.