# Operating Systems

## Multithreading:
Multithreading in Java is a programming technique that allows multiple threads to run concurrently, sharing the same process. This is particularly useful for performing tasks simultaneously, such as executing background tasks, responding to user input, or improving the performance of computationally intensive programs. 

- Multithreading offers performance benefits but also introduces several challenges:
    - ****Race Conditions:**** Occur when multiple threads access shared data concurrently. Overcome using synchronization mechanisms like locks.
    - ****Deadlocks:**** Threads get stuck waiting for each other. Avoid by ordering resource acquisition or using timeouts.
    - ****Synchronization Overhead:**** Excessive synchronization can slow performance. Minimize critical sections and use thread pools (`ExecutorService`).
    - ****Context Switching:**** The cost of switching between threads can reduce performance. Limit the number of threads and keep them busy.
    - ****Starvation:**** is a situation in multithreading where a thread is perpetually denied access to the resources it needs to execute, often because other threads are continuously given priority.
    - ****Livelock:**** In a livelock, two threads are actively trying to avoid deadlock but continually fail to make progress because they keep changing their states without performing useful work.

- **[Process](https://www.geeksforgeeks.org/process-in-operating-system/)**: A process is a complete execution of a task which has its privite memory space and resources. A process can have multiple threads.
- **Thread:** A thread is a part of the process which has its run-time stack and shares the process resources. Every thread in Java is created and controlled by a unique object of the java.lang.Thread class. When a standalone application is run, a user thread is automatically created to execute the main() method. This thread is called the main thread.
- **Thread Lifecycle:** [See here](https://www.geeksforgeeks.org/lifecycle-and-states-of-a-thread-in-java/)
- **Thread Priority:** [See here](https://www.geeksforgeeks.org/java-thread-priority-multithreading/)
- Thread creation with `Runnable` Interface:

  ```java
      public class RunnableThread {
        public static int count = 0;
  
        public static void main(String[] args) {
  
            Thread thread = new Thread(() -> {
                System.out.println("RunnableThread starting.");
                try {
                    while (RunnableThread.count < 5) {
                        Thread.sleep(500);
                        RunnableThread.count++;
                    }
                } catch (InterruptedException exc) {
                    System.out.println("RunnableThread interrupted.");
                }
                System.out.println("RunnableThread terminating.");
            });

            System.out.println("Executing start()");
            thread.start();
        }
    }
  ```
  
- **Concurrency:** The ability of a system to handle multiple tasks simultaneously, but not necessarily at the same time.
- **Parallelism:** The ability to execute multiple tasks at the same time, typically across multiple cores of a processor.
- **Synchronization:** Synchronization in Java is a mechanism used to control access to shared resources by multiple threads. It ensures that only one thread can access a `synchronized` block or method at a time, preventing thread interference and memory consistency issues.
- **Locks:** Locks in Java provide a more flexible mechanism for thread synchronization than traditional synchronization blocks or methods. Unlike intrinsic locks (synchronized), explicit locks provide better control over thread behavior and offer advanced features like fairness policies and condition variables. The `ReentrantLock` is the most commonly used lock in Java. It behaves similarly to synchronized blocks but offers additional features. `ReadWriteLock` and `StampedLock` are other types of locks.

#### **Example: Using ReentrantLock**

```java
import java.util.concurrent.locks.ReentrantLock;

class SharedResource {
    private ReentrantLock lock = new ReentrantLock();

    void print(String message) {
        lock.lock(); // Acquire the lock
        try {
            System.out.print("[");
            System.out.print(message);
            System.out.println("]");
        } finally {
            lock.unlock(); // Release the lock
        }
    }
}

class MyThread extends Thread {
    SharedResource resource;
    String message;

    MyThread(SharedResource resource, String message) {
        this.resource = resource;
        this.message = message;
    }

    public void run() {
        resource.print(message);
    }
}

public class ReentrantLockExample {
    public static void main(String[] args) {
        SharedResource resource = new SharedResource();
        MyThread t1 = new MyThread(resource, "Hello");
        MyThread t2 = new MyThread(resource, "World");

        t1.start();
        t2.start();
    }
}
```
---

#### Deadlocks and Deadlock Prevention:
A deadlock is a situation where a thread is waiting for an object lock that another thread holds, and this second thread is waiting for an object lock that the first thread holds (or an equivalent situation with several threads). Since each thread is waiting for the other thread to relinquish a lock, they both remain waiting forever. The threads are said to be deadlocked.
- In order for a deadlock to occur, you must have all four of the following conditions met:
    - **Mutual Exclusion:** Only one process can access a resource at a given time. (Or, more accurately, there is limited access to a resource. A deadlock could also occur if a resource has limited quantity.)
    - **Hold and Wait:** Processes already holding a resource can request additional resources, without relinquishing their current resources.
    - **No Preemption:** One process cannot forcibly remove another process' resource.
    - **Circular Wait:** Two or more processes form a circular chain where each process is waiting on another resource in the chain.
- Deadlock prevention entails removing any of the above conditions, but it gets tricky because many of these conditions are difficult to satisfy. For instance, removing #1 is difficult because many resources can only be used by one process at a time (e.g., printers). Most deadlock prevention algorithms focus on avoiding condition #4: circular wait.

### Dining Philosophers Problem:
The **Dining Philosophers Problem** is a classic synchronization problem in computer science, where several philosophers are sitting at a round table, thinking and occasionally eating. To eat, a philosopher needs two forks (one on the left and one on the right). However, if two adjacent philosophers pick up the same fork simultaneously, they could end up deadlocked, starving, or conflicting.

#### Approach
1. **Shared resources (forks)**: Each fork is a shared resource between two adjacent philosophers.
2. **Locks**: Use locks (Java's `ReentrantLock`) to ensure mutual exclusion while picking up and putting down the forks.
3. **Philosophers as threads**: Each philosopher is modeled as a thread that alternates between thinking and eating.

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class DiningPhilosophers {

    // Number of philosophers
    private static final int NUM_PHILOSOPHERS = 5;

    // Array of forks represented by Locks
    private static final Lock[] forks = new Lock[NUM_PHILOSOPHERS];

    // Initialize the forks (locks)
    static {
        for (int i = 0; i < NUM_PHILOSOPHERS; i++) {
            forks[i] = new ReentrantLock();
        }
    }

    public static void main(String[] args) {
        // Create and start philosopher threads
        for (int i = 0; i < NUM_PHILOSOPHERS; i++) {
            final int philosopherId = i;
            new Thread(new Philosopher(philosopherId)).start();
        }
    }

    static class Philosopher implements Runnable {
        private final int id;

        public Philosopher(int id) {
            this.id = id;
        }

        @Override
        public void run() {
            try {
                while (true) {
                    think();
                    // Pick up forks in order: first left, then right
                    pickUpForks();
                    eat();
                    // Put down forks after eating
                    putDownForks();
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }

        private void think() throws InterruptedException {
            System.out.println("Philosopher " + id + " is thinking.");
            Thread.sleep((long) (Math.random() * 1000)); // Thinking for a while
        }

        private void eat() throws InterruptedException {
            System.out.println("Philosopher " + id + " is eating.");
            Thread.sleep((long) (Math.random() * 1000)); // Eating for a while
        }

        private void pickUpForks() {
            // Pick up the left fork (id) and right fork ((id + 1) % NUM_PHILOSOPHERS)
            Lock leftFork = forks[id];
            Lock rightFork = forks[(id + 1) % NUM_PHILOSOPHERS];

            // Try to lock the forks, avoid deadlock by always locking the lower-index fork first
            if (id % 2 == 0) {
                leftFork.lock();
                rightFork.lock();
            } else {
                rightFork.lock();
                leftFork.lock();
            }
        }

        private void putDownForks() {
            // Put down the forks after eating
            Lock leftFork = forks[id];
            Lock rightFork = forks[(id + 1) % NUM_PHILOSOPHERS];

            leftFork.unlock();
            rightFork.unlock();
        }
    }
}
```
---