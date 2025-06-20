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
```

```

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


```
public class EvenOddPrinter {
    int count = 1;
    final int maxValue = 10;

    public static void main(String[] args) {
        EvenOddPrinter printer = new EvenOddPrinter();
        Thread oddThread = new Thread(() -> printer.printOdd(), "Odd Thread");
        Thread evenThread = new Thread(() -> printer.printEven(), "Even Thread");
        oddThread.start();
        evenThread.start();
    }

    public synchronized void printOdd() {
        while (count <= maxValue) {
            if (count % 2 == 0) {
                try {
                    wait();
                } catch (InterruptedException exception) {
                    Thread.currentThread().interrupt();
                }
            }
            else{
                System.out.println(Thread.currentThread().getName()+"-"+ count++);
                notify();
            }
        }
        notify();
    }

    public  synchronized void printEven() {
        while (count <= maxValue) {
            if (count % 2 != 0) {
                try {
                    wait();
                } catch (InterruptedException exception) {
                    Thread.currentThread().interrupt();
                }
            }
            else{
                System.out.println(Thread.currentThread().getName()+"-"+ count++);
                notify();
            }
        }
        notify();
    }
}
```
### join()

The join() method in Java is used to pause the execution of the current thread until the thread on which join() was called has finished executing.


### ✅ Thread Priority in Java
In Java, thread priority is a hint to the thread scheduler about the importance of a thread. It does not guarantee execution order, but influences which threads the scheduler may choose to run.

### 🔢 Priority Range
Java threads have a priority value from 1 to 10:

Constant	Value	Description
Thread.MIN_PRIORITY	1	Lowest priority
Thread.NORM_PRIORITY	5	Default priority
Thread.MAX_PRIORITY	10	Highest priority


###  Callable and Runnable in Java

Both Runnable and Callable are used to define tasks that can be executed by threads, but they have key differences:

### ✅ Callable vs Runnable – Key Differences
 #### Return Value:

Runnable: Cannot return a result (run() returns void).

Callable: Can return a result (call() returns a value of type V).

#### Exception Handling:

Runnable: Cannot throw checked exceptions.

Callable: Can throw checked exceptions.

#### Method to Implement:

Runnable: Must implement run() method.

Callable: Must implement call() method.

#### Introduced In:

Runnable: Available since Java 1.0.

Callable: Introduced in Java 5 (part of java.util.concurrent package).

#### Usage with Threading API:

Runnable: Used with Thread or ExecutorService.

Callable: Used only with ExecutorService and returns a Future.

#### Getting Result:

Runnable: Cannot provide any result.

Callable: Returns a Future from which the result can be fetched using future.get().

#### Use Case:

Runnable: Suitable for tasks where no result is required.

Callable: Suitable when you need the result of a task or need to handle checked exceptions.





