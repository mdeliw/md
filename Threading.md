# Threading

## CountDownLatch

https://www.baeldung.com/java-countdown-latch

We can cause a thread to block until other threads have completed a given task. 

- A counter is set to represent the number of tasks or number of threads the blocked thread needs to wait for. 
- When each task is complete, the counter is decremented by 1.
- The counter is used to block a thread until its counted down to zero.