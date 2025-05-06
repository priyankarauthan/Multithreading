# Multithreading


## 1. What is Multithreading?
Multithreading is a feature in Java that allows multiple parts of a program (threads) to run concurrently, improving performance and responsiveness.

Key Benefits:
✅ Better CPU Utilization – Multiple threads can use the CPU efficiently.
✅ Faster Execution – Tasks can be executed in parallel.
✅ Improved Responsiveness – Useful for applications like GUI, servers, and real-time systems.

## 2. Creating Threads in Java
Java provides two ways to create a thread:

Method 1: Extending Thread class
```
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread is running...");
    }
    
    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        t1.start(); // Start the thread
    }
}
```
Method 2: Implementing Runnable interface (Preferred way)
```
class MyRunnable implements Runnable {
    public void run() {
        System.out.println("Thread is running...");
    }
    
    public static void main(String[] args) {
        Thread t1 = new Thread(new MyRunnable());
        t1.start(); // Start the thread
    }
}
```
#### ✅ Why use Runnable?

Java supports single inheritance, so extending Thread prevents extending another class.

Runnable is more flexible and widely used.

### 3. Thread Lifecycle
A thread goes through several states:

New – Created but not started (new Thread()).

Runnable – Ready to run (start() is called).

Running – Thread is executing.

Blocked/Waiting – Thread is waiting for a resource.

Terminated – Thread has finished execution.

### 4. Thread Methods
   
Method	Description

start()	Starts a new thread.

run()	Contains the code to be executed.

sleep(ms)	Puts thread to sleep for a given time.

join()	Waits for a thread to complete execution.

yield()	Suggests giving other threads a chance to execute.

isAlive()	Checks if the thread is still running.

Example using join():

```
class MyThread extends Thread {
    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.println(i);
            try { Thread.sleep(1000); } catch (InterruptedException e) {}
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        MyThread t1 = new MyThread();
        t1.start();
        t1.join(); // Main thread waits for t1 to finish
        System.out.println("Thread execution completed.");
    }
}
```
5. Synchronization (Handling Race Conditions)
When multiple threads access shared resources, synchronization is needed to prevent race conditions.

Example of a Race Condition:
```
class Counter {
    int count = 0;
    
    public void increment() {
        count++;
    }
    
    public static void main(String[] args) {
        Counter c = new Counter();
        
        Thread t1 = new Thread(() -> { for (int i = 0; i < 1000; i++) c.increment(); });
        Thread t2 = new Thread(() -> { for (int i = 0; i < 1000; i++) c.increment(); });

        t1.start();
        t2.start();
        
        try { t1.join(); t2.join(); } catch (InterruptedException e) {}

        System.out.println("Final count: " + c.count); // May not be 2000 due to race conditions!
    }
}
```
Fixing with Synchronization:
```
class Counter {
    int count = 0;

    public synchronized void increment() { // Synchronized method
        count++;
    }
}
```
### 6. Executor Framework (Better Thread Management)
Java provides the Executor Framework for managing thread pools.

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

class MyTask implements Runnable {
    public void run() {
        System.out.println(Thread.currentThread().getName() + " is executing.");
    }
    
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(3); // Create a pool of 3 threads
        
        for (int i = 0; i < 5; i++) {
            executor.execute(new MyTask());
        }
        
        executor.shutdown(); // Shutdown the executor
    }
}
```
## ✅ Why use Executors?

Avoids creating too many threads.

Manages thread lifecycles efficiently.


#### Producer-Consumer Problem

```
public class ProducerConsumerExample {
    public static void main(String[] args) {
        Buffer buffer = new Buffer();
        Thread producerThread = new Thread(new Producer(buffer));
        Thread consumerThread = new Thread(new Consumer(buffer));

        producerThread.start();
        consumerThread.start();
    }
}

class Producer implements Runnable {
    private final Buffer buffer;

    public Producer(Buffer buffer) {
        this.buffer = buffer;
    }

    public void run() {
        int item = 0;
        while (true) {
            try {
                buffer.produce(item++);
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class Consumer implements Runnable {
    private final Buffer buffer;

    public Consumer(Buffer buffer) {
        this.buffer = buffer;
    }

    public void run() {
        while (true) {
            try {
                buffer.consume();
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}



import java.util.LinkedList;
import java.util.Queue;

class Buffer {
    private final int capacity = 5;
    private final Queue<Integer> queue = new LinkedList<>();

    public synchronized void produce(int item) throws InterruptedException {
        while (queue.size() == capacity) {
            System.out.println("Buffer full. Producer is waiting...");
            wait(); // wait until buffer has space
        }
        queue.add(item);
        System.out.println("Produced: " + item);
        notify(); // notify consumer that item is available
    }

    public synchronized int consume() throws InterruptedException {
        while (queue.isEmpty()) {
            System.out.println("Buffer empty. Consumer is waiting...");
            wait(); // wait until buffer has items
        }
        int item = queue.poll();
        System.out.println("Consumed: " + item);
        notify(); // notify producer that space is available
        return item;
    }
}

```


## Monitor Lock
