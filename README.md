# Lab 01: Multithreading & Concurrency

## Task 2: Race Conditions & Thread Safety

### 1. UnsafeCounter Race Condition Trials
Running 8 threads with 1,000,000 increments each (Expected: 8,000,000):
- **Run 1**: 2,223,384 (correct: false)
- **Run 2**: 1,894,201 (correct: false)
- **Run 3**: 3,105,449 (correct: false)

### 2. Why is LongAdder faster under high contention?
LongAdder maintains an internal array of cell variables. When multiple threads try to update the counter simultaneously, LongAdder routes different threads to different cells, avoiding CPU cache-line bouncing and lock/CAS contention. The total result is computed by summing the cells when sum() is called.

### 3. Why does making only increment() synchronized but NOT get() leave a bug?
Without synchronizing get(), the Java Memory Model does not establish a **happens-before** relationship between the thread calling increment() and the thread calling get(). As a result, reading threads may see stale cached values.

---

## Task 3: Producer-Consumer with wait()/notifyAll() and BlockingQueue

### 1. Changing while to if in Guarded Blocks
- **Defect**: Spurious wakeups or state changes by another thread.
- **Why it breaks**: If a consumer thread wakes up from wait(), an if block checks the condition only once. If another consumer thread grabbed the item first, the woken consumer proceeds believing the queue is non-empty, leading to NoSuchElementException on queue.remove().

### 2. Changing 
otifyAll() to 
otify()
- **Defect**: Deadlock / Signal Loss.
- **Why it breaks**: 
otify() wakes up only one arbitrary waiting thread. If a Producer finishes putting an order and calls 
otify(), Java might wake up another *Producer* that is waiting on a full buffer instead of a *Consumer*. That producer sees the buffer is still full and goes back to sleep, leaving all threads permanently waiting in deadlock.

---

## Task 4: Thread Pools, Future, and Virtual Threads

### 1. Platform Threads vs. Virtual Threads Performance Difference
- **Platform Pool (20 threads)**: Processing 500 tasks with a 200 ms blocking sleep takes approximately **5,000 ms** (5 seconds). Because only 20 threads are available, tasks are bottlenecked in a queue waiting for worker threads to finish sleeping.
- **Virtual Threads (
ewVirtualThreadPerTaskExecutor)**: Processing 500 tasks takes approximately **1,000–1,200 ms** (governed mostly by the 1-second timeout task). Virtual threads unmount from carrier threads during blocking I/O calls (Thread.sleep), enabling thousands of concurrent operations with negligible overhead.

### 2. Error and Timeout Handling Summary
- **ExecutionException**: Successfully catches underlying task exceptions (such as IOException("boom")) without crashing the application pool.
- **TimeoutException**: Caught using uture.get(1, TimeUnit.SECONDS). Calling uture.cancel(true) safely interrupts long-running blocking tasks.