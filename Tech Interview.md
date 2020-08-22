Tech Interview Questions

[toc]

# Vertx

## Mixed Threads

- Integrate vert.x with a third-party software which has its own threading mechanism. How would you design a verticle that offloads to a non-vertx thread and gains control back on vertx event loop? 

```java
public class SampleVerticle extends AbstractVerticle {
  private final Logger logger = LoggerFactory.getLogger(SampleVerticle.class);
  public static void main(String[] args) {
    Vertx vertx = Vertx.vertx();
    vertx.deployVerticle(new SampleVerticle());
  }

  @Override
  public void start() throws Exception {
    logger.info("I am starting a verticle - on event-loop");
    Context context = vertx.getOrCreateContext();
    new Thread( () -> {
      try {
        run(context);
      } catch (InterruptedException e) {
        logger.error("Woops", e);
      }
    }).start();
  }

  private void run(Context context) throws InterruptedException {
    logger.info("I am in a non-vertx thread");
    CountDownLatch latch = new CountDownLatch(1);
    context.runOnContext(v -> {
      logger.info("I am on the event-loop");
      vertx.setTimer(1000, id -> {
        logger.info("This is the final countdown");
        latch.countDown();
      });
    });
    logger.info("Waiting on the countdown latch ...");
    latch.await();
    logger.info("Bye!");
  }
}
```



# Concurrency

## Java 8

### Data Structure Convesion

- Create a Stream of Integers

### Questions

- Say you have a `Runnable` class `Worker` defined. How can you create 5 threads and launch them using `Streams`?

```java
List<Thread> workers = Stream
    .generate(() -> new Thread(new Worker()))
    .limit(5)
    .collect(toList);

workers.forEach(Thread::start);
```

## CountDownLatch

### Questions

- Explain CountDownLatch clearly
- What happens when a thread errors out without counting down?

### Problems

https://www.baeldung.com/java-countdown-latch

- Use it to allow a parent thread to block until  all its children threads are complete.
- Use it to allow the children threads to block so that they can start at the same time, and the parent thread to block until all its children threads are complete.