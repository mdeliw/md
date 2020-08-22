# Interview

[toc]

## Concurrency

| Category | Qs                        | Answer                                                       |
| -------- | ------------------------- | ------------------------------------------------------------ |
| Junior   | Thread state              | New, runnable, running, waiting, timed_wait, terminated      |
|          | User, Daemon thread       | JVM waits for user thread to complete. Daemon are used by User threads. |
|          | Monitor or Intrinsic Lock | One per object, thread owns the monitor when it enters the synchronized block. Can release the monitor temporarily via object.wait |
|          | Semaphore                 | signaling mechanism for locking control: wait, signal. Binary and Counting Semaphore |
|          | Mutex                     | Is a binary semaphore. Mutex is better because? Unlike Semaphore, the thread acquiring the mutex is the only thread that can release the mutex. |
|          | Java Lock                 | Lock, Unlock for synchronized blocks, ReadWriteLock          |
| Advanced | CLI for Thread size       | `java -XX:+PrintFlagsFinal -version`                         |



