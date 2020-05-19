# Concurrency in Java

## Basic concepts

A *thread* is the smallest unit of execution that can be scheduled by the operating system.

A *process* is a group of associated threads that execute in the same shared environment.

A *shared environment* is a memory space where threads can communicate with each other.

Static methods and variables are particularly useful for threads because there is a single class object all instances share.  
Therefore if one thread updates something static, it is immediately available for other threads.

All java applications are multi threaded. *System threads* are created by the JVM and run in the background.  
System threads generate Errors rather than Exceptions. E.g. OutOfMemoryError.

*Concurrency* is the concept of executing multiple threads or processes at the same time. 
When operating on a single core (or multi core!) a *thread schedular* determnies whhiich threads should be executing. 
This is because only one task can be executed at a time on a processor.
A *context switch* is the process of storing a thread's current state so it can be reloaded.
*Thread priority*  is used to determine which threads should currently be executing.

In Java thread priority ranges from 1 (min) - 10 (max). 5 is normal priority.

## Simple implementation

### Introducing Runnable
``` java
@FunctionalInterface public interface Runnable {
    void run();
}
....
public class CalculateAverage implements Runnable {
    public void run() {
        // define work here
    }
}
```
Functional interfaces take no arguments and return no data, and the above one is commonly used to define the work of a thread.

### Creating a Thread
Executing a thread involves 1. Defining the thread and task to be done  2. Starting the task by ``Thread.start()``
``` java
public static void main(String[] args){
    (new Thread(new CalculateAverage())).start();
}
```
The thread class can also be extended and ``run()`` overriden.  
However note that implementing runnable is the more common approach as only one class can be extended in Java (but many interfaces can be implemented)

What will the below print?

``` java
public class PrintNumbers implements Runnable {
    public void run() {
        System.out.println(1);
        System.out.println(2);
        System.out.println(3);
    }
}
....
public class PrintLetters implements Runnable {
    public void run() {
        System.out.println("a");
        System.out.println("b");
        System.out.println("c");
    }
}
...
public static void main(String[] args){
    System.out.println("begin");
    (new Thread(new PrintNumbers())).start();
    (new Thread(new PrintLetters())).start();
    System.out.println("end");
}
```
```
begin
1
a
end
2
3
b
c
```

The answer isn't known until runtime. Something else could have been printed.  
The order within a thread is linear, but the order of thread execution is *indeterminate*.

GOTCHA:
The below will actually execute in order. ``run()`` doesn't start a separate processing thread.
``` java
public static void main(String[] args){
    System.out.println("begin");
    (new Thread(new PrintNumbers())).run();
    (new Thread(new PrintLetters())).run();
    System.out.println("end");
}
```
```
begin
1
2
3
a
b
c
end
```

### Polling with Sleep

*Polling* is the process of intermittently checking data at some fixed interval. 

This sounds like a while loop! However using a ``while(..)`` loop in a thread scheduler is consider bad practise.  

The while loop could operate infinitely!

``Thread.sleep()`` can be used to implement polling. It throws the checked ``InterruptedException``.

``` java
public class CheckResults {
    private static int counter = 0;

    public static void main(String[] args) throws InterruptedException {

        new Thread(() -> {
            for(int i = 0; i < 500; i++) CheckResults.counter++;
        }).start();

        while(CheckResults.counter < 100) {
            System.out.println("Not reached yet");
            Thread.sleep(1000); // this is the polling
        }
    }

}
```

The above implements polling and does prevent the CPU from being overwhlemed with a potentially infinite loop. 
However it does not guarantee when the loop will terminate.

## Using Executor services

The ``ExecutorService`` creates and manages threads for you.
``SingleThreadExectuor`` is the simplest ExecutorService and guarantees results are executed in the order in which they were added to the executor service.
Example with letters and numbers printing:

``` java
public class Printer {
    public static void main(String[] args) {
        ExecutorService service = null;
        try{
            service = Executors.newSingleThreadExectuor();
            System.out.println("begin");
            service.execute(() -> {
                System.out.println("a");
                System.out.println("b");
                System.out.println("c");
            });
            service.execute(() -> {
                System.out.println(1);
                System.out.println(2);
                System.out.println(3);
            });
            System.out.println("end");
        } finally {
            if(service != null) service.shutdown();
        }
        System.out.println("end 2");
    }
}
```
```
begin
a
b
end
c
1
2
end 2
3
```
is one of the options that could be printed ('end' and 'end 2' could be somewhere else)

Note that end 2, outside of the try/catch could be printed while the threads are still executing - the program continues after the threads have been set off.

It is important to call the ``shutdown()`` method once a thread has finished executing. Otherwise your application will never terminate.  

In real world examples a finally block is used as above for this.  

While shutting down ``isTerminated()`` can be called to see if the thread shutdown has been completed. 

## Submit and Future object
 An alternative to ``.execute()`` is ``.submit()`` which returns a ``Future<V>`` object that can be used to determine whether or not a task has completed execution.

 Some useful methods on Future:
 * ``boolean isDone()`` - Returns true if task completed, threw exception, or cancelled
 * ``boolean isCancelled()``
 * ``boolean cancel()`` - Attempts to cancel task
 * ``V get()`` - Retrieves the result of a task, waiting **ENDLESSLY** for it.
 * ``V get(long timeout, TimeUnit unit)`` - As above but if result not ready by ``timeout`` throws a ``TimeoutException``

 The last one is very commonly used for *polling*, but the below example never explicitly uses the ``Thread`` class. This is the essence of the Concurrency API.

 ``` java
public class CheckResults {
    private static int counter = 0;

    public static void main(String[] args) throws InterruptedException, ExecutionException {

        ExecutorService service = null;
        try{
            service = Executors.newSingleThreadExectuor();

            Future<?> result = service.submit(() -> {
                for(int i = 0; i < 500; i++) CheckResults.counter++;
            });

            result.get(10, TimeUnit.SECONDS);
            System.out.println("Reached!");
        } catch (timeoutException e) {
            System.out.println("Not reached in time");
        }
        } finally {
            if(service != null) service.shutdown();
        }
    }

}
 ```

 ## Callable

 Callable is like Runnable but returns a value.

 ``` java
 @FunctionalInterface public interface Callable {
    V call() throws Exception;
}
 ```

 The ``ExecutorService`` includes an overload version of ``submit()`` that takes a ``Callable`` object and returns a generic ``Future<V>`` object.

 On a ``Future``, ``get()`` returns null on a ``Runnable``, and the matching generic type or null on a ``Callable``.

 ``` java
public class AddData {
    public static void main(String[] args) throws InterruptedException, ExecutionException {

        ExecutorService service = null;
        try{
            service = Executors.newSingleThreadExectuor();

            Future<?> result = service.submit(() -> 30 + 11);
            System.out.println(result.get()); //prints 41
        } finally {
            if(service != null) service.shutdown();
        }
    }

}
 ```

 ## Waiting for tasks to finish
 A seen previously ``get()`` on a Future can be used to wait for results.  
 However if we don't need the results of the tasks and are finished using our thread executor there is a simpler approach:

``` java
ExecutorService service = null;
try{
    service = Executors.newSingleThreadExectuor();
    // Add threads
} finally {
    if(service != null) service.shutdown();
}

if(service != null) {
    service.awaitTermination(1, TimeUnit.MINUTES);
    // Check whether all tasks are finished
    if(service.isTerminated()){
        System.out.println("All tasks finished");
    } else {
        System.out.println("At least one task is still running");
    }
}
```
``awaitTermination()`` is called before ``isTerminated()``.

## Scheduling Tasks
The ``ScheduledExectuorService`` can be used for tasks that occur periodically.

``` java
ScheduledExecutorService service = Executors.newSingleThreadScheduledExecutor();

Runnable task1 = () -> Systemoput.println("Hello Zoo");
Callable<String> task2 = () -> "Monkey";

Future<?> result1 = service.schedule(task1, 10, TimeUnit.SECONDS);
Future<?> result2 = service.schedule(task2, 8 , TimeUnit.MINUTES);
```
task1 will happen 10 seconds in the future.  
task2 will happen 8 minutes in the future.

For ``Runnable`` only ``scheduleAtFixedRate(...)`` and ``scheduleAtFixedDelay(...)`` are also possible.
 
ScheduleAtFixedDelay ensures creates a new task after the previous task has finished.  
This prevents a potential problem with scheduleAtFixedRate where a new task starts before the last has finished.  

Otherwise ScheduleAtFixedRate is the closest Java has to a cronjob.

## Thread Pools

A *thread pool* is a group of pre-instantiated reusable threads that are available to perform a set of arbitrary tasks.

Some types of ThreadPool:
* ``newCachedThreadPool()`` - returns an ``ExecutorService`` - TP that creates new threads as needed, but will reuse previos threads when available.
* ``newFixedThreadPool(int nThreads)`` - returns an ``ExecutorService`` - TP that reuses a fixed number of threads operating off a shared unbounded queue.
* ``newScheduledThreadPool(int nThreads)`` - returns a ``ScheduledExecutorService`` - TP that can schedule commands to run after a given delay or execute periodically.

Single thread executors wait for an available thread to become available before running the next task.
Pooled thread executors can execute the next task concurrently, and if the pool runs out of available thread, the task will be queued by the thread executor and wait to be completed.

Otherwise our previous example are compatible with pooled thread executors.

Some health notes:
``newCachedThreadPool()`` are often used for short lived asynchronous tasks. For long lived processes they are strongly discouraged.
``newFixedThreadPool(1)`` is just a ``newSingleThreadExecutor()``.

It is common practise to assign the maximum number of long lived threads to the number of available CPUs:
``Runtime.getRuntime().availableProcessors()``

## Synchronizing Data Access
Now that we have multiple threads capable of accessing the same objects in memory, we have to make sure to organize our access to this data correctly.  
How do we prevent 2 threads from interfering with each other?

The unexpected result of two tasks executing at the same time is called a *race condition*.

In Java ``Atomic`` classes solve this problem. Thread safe atomic classes only allow reading and writing as a single operation.

This doesn't allow any other thread to access the variable during the operation.

Operations like this include ``get()``, ``getAndIncrement()``, ``getAndSet()``.

There are primitive equivalent classes like ``AtomicInteger`` but also collection ones suchas ``AtomicIntegerArray``.

*Here's a counting example without Atomic:*

``` java
public class Counter {
    
    private int count = 0;

    private void incAndPrint() {
        System.out.println(++count);
    }

    public static void main(String[] args) {
        ExecutorService service = null;
        try{
            service = Executors.newFixedThreadPool(20);
            Counter counter = new Counter();
            for(int i= 0; i < 10; i++){
                service.submit(() -> counter.incAndPrint());
            }
        } finally {
            if(service != null) service.shutdown();
        }
    }
}
```
Will print duplicates and could miss values (but will print 10 values):
```
1 2 2 3 4 5 6 7 8 9
2 4 5 6 7 8 1 9 10 3
```

*Now adding in an Atomic integer:*
``` java
...
    private AtomicInteger count = new AtomicInteger(0);

    private void incAndPrint() {
        System.out.println(count.incrementAndGet());
    }
...
```
Will print no duplicates and miss no values, but the results might not be ordered:
```
2 3 1 4 5 6 7 8 9 10
1 4 3 5 6 2 7 8 10 9
```

Atomic ensures data is consistent between workers and no values are lost due to concurrent modifications.

To make results are reported back in order we can use a *lock* with the ``synchronized`` keyword.

Using this in the above example would make the following replacements:

``` java
    private int sheepCount = 0; //Atomic not needed with synchronized here 

    private void incAndPrint() {
        synchronized(this){
            System.out.println(++count);
        }
    }
```
which will print the following :)
```
1 2 3 4 5 6 7 8 9 10
```
The threads are still created and executed at the same time, but each waits at the ``synchronized`` block for the previous worker before entering.

Java provides another way of writing this as well:
``` java
private synchronized void incAndPrint(){
    System.out.println(++count);
}
```

The synchronized keyword can also be added to static methods.

*What is the cost of synchronization??*
Multi threaded programming is about taking multiple threads and making them do multiple things at the same time. Synchronization is about taking multiple threads and making them perform in a more single-threaded manner. In extreame situations synchronization could significantly slow down an application.

## Concurrent Collections

There are alternatives to the core Java collection classes to prevent you from having to do something like this and making mistakes with your custom implementation.

``` java
private Map<String, Object> map = new HashMap<String, Object>();

public synchronized void put(String key, Object value){
    map.put(key, value);
}
```

now we can just have:

``` java
private Map<String, Object> map = new ConcurrenHashMap<String, Object>();
map.put(..., ...);
```
Using these lower our risks of a ``ConcurrentModificationException``.

There are also method for obtaining synchronized versions of existing non-concurrent collections.

e.g.
``` java
List<Integer> list = Collections.synchronizedList(New ArrayList<>(Arrays.asList(4, 3, 52)));
```

A couple of the more interesting ones to understand:

### Blocking Queues

``LinkedBlockingQueuee`` inheritr all method from queue but also has methods that will wait a specific amount of time to complete an operation.

``.offer(E e, long timeout, TimUnit unit)`` - add item to queue waiting the specified time returning false if time elapses before space is available.

``.poll(long timeout, TimeUnit unit)`` - retrieves and removes an item from the queue waiting the specified time and returning null if the time elapses before the item is available.

These methods can throw an ``InterruptedException`` as they can be interrupted before they finish waiting for a result.

### SkipList Collections
``ConcurrentSkipeListSet`` and ``ConcurentSkipListMap`` are concurrent versrions of ``TreeSet`` and ``TreeMap``.

## CopyOnWritCollections
``CopyOnWriteArrayList`` and ``CopyOnWriteArraySet`` behave a bit differently to some of the above. Any time an lmnt is modfied they copy all their elements to a new underlying structure. Our reference to the object does not change.

One interesting thing this allows is a different iteration behaviour:

``` java
List<Integer> list = new CopyOnWriteArrayList<>(Arrays.asList(4, 3 , 52));
for (Integer x: list){
    System.out.println(x)
    list.add(1);
}
System.out.println("Size: " + list.size());
```
```
4
3
52
Size: 6
```

WARNING: these classes can use a lot of memory. They are commonly used in multi-threaded environments where reads are far more common than writes.

## Parallel Streams

A *serial stream* is a stream where the results are ordered and processed one at a time.

A *parallel stream* is a streaem that is capable of processing results concurrently using multiple threads.

They can be created as so:

``` java
Stream<Integer> parallelStream = stream.parallel(); //method 1
Stream<Integer> parallelStream2 = Arrays.asList(1,2,3,4,5,6).parallelStream(); //method 2
```

comparing simple streams:

``` java
Arrays.asList(1,2,3,4,5,6)
    .stream()
    .forEach(s -> System.out.println(s + " ");)
```
```
1 2 3 4 5 6
```
``` java
Arrays.asList(1,2,3,4,5,6)
    .parallelStream()
    .forEach(s -> System.out.println(s + " ");)
```
```
6 2 3 1 5 4
```
Could be any other order!
``` java
Arrays.asList(1,2,3,4,5,6)
    .parallelStream()
    .forEachOrdered(s -> System.out.println(s + " ");)
```
```
1 2 3 4 5 6
```
``forEachOrdered`` forces ordering in that specific part of the parallel stream.

This has obvious performance losses but is really useful in allowing part of the stream to be ordered while other parts can still get the performance imporvements of threading.
Some things to note:
* *Stateful lambda expressions* should be avoided. These are ones where the result depends on any state that might change during the execution of the pipeline.
* *Order based tasks* can behave weirdly. e.g. ``.findAny()`` consistently outputs the first value in the serial stream equivalent of the parallel stream. (i.e. 1 in [1, 2, 3]). Any stream operation that is based on order including ``.findFirst()``, ``.limit()``, ``skip`` will return consistently with that of a serial stream but may **perform slower**.
* Methods like ``reduce()`` and ``collect()`` require additional parameters in parallel streams -  *identity*, *accumulator* and *combiner*.

## Managing Concurrent Processes

### Cyclic Barriers
A cyclic barrier allows threads to pause and wait for each other at the end of a task/ set of tasks before starting on the next bit of work.

Here's an example where we have 4 zoo workers who we want to work together to clean an animal pen.

``` java
public class LionPenManager {

    private void removeAnimals() {System.out.println("Removing animals");}
    private void cleanPen(){System.out.println("Cleaning the pen");}
    private void addAnimals(){System.out.println("Adding animals");}

    public void performTask(CyclicBarrier c1, CyclicBarrier c2){
        try {
            removeAnimals();
            c1.await();
            cleanPen();
            c2.await();
            addAnimals();
        } catch (INterruptedExceptio | BrokenBarrierException e){
            // handle
        }
    }

    public void static main(String[] args) {
        ExecutorService service = null;
        try {
            service = Execturos.newFixedThreadPool(4);

            LionPenManager manager = new LionPenManager();
            CyclicBarrier c1 = new CyclicBarrier(4, 
                () -> System.out.println"*** Animals Removed!"));
            CyclicBarrier c2 = new CycliceBarrier(4,
                () -> System.out.println"*** PenCleaned!"));
            for(int i=0; i<4; i++){
                service.submit(() -> manager.performTask(c1, c2));
            } finally {
                if(service != null) service.shutdown();
            }
        }
    }
}
```
```
Removing animals
Removing animals
Removing animals
Removing animals
*** Animals Removed!
Cleaning the pen
Cleaning the pen
Cleaning the pen
Cleaning the pen
*** PenCleaned!
Adding animals
Adding animals
Adding animals
Adding animals
```

If using a thread pool. The number of available threads must be at least as large as the CyclicBarrier limit value.  Otherwise the code will hang indefinitely!

A slight loss of performance can be expected from using a CyclicBarrier as one slow may be much slower and the rest must wait for it.

**Reuse: if we have 15 threads that call ``await()`` and the limit of the ``CyclicBarrier`` is 5 the cyclic barrier will be activated 3 times.**
### Fork/ Join

When a task gets too complicated we can split the task into multiple other tasks using fork/join which will determine how many threads to create. Fork/Join use the concept of recursion where the task calls itself.

The key classes for fork/join are ``RecursiveAction`` and ``RecursiveTask`` which implement the ``ForkJoinTask`` interface.

Both classes are abstract, and require implementation of ``compute()`` but ``RecursiveTask`` is also generic and ``compute()`` returns a generic wheras for ``RecursiveAction`` it returns ``void``. They are analagous to ``Runnable`` and ``Callable``.

Here is another Zoo based example. There are 10 animals to weight in an hour, but each zookeeper can only weigh 3 animals an hour.

``` java
public class WeighAnimalAction extends RecursiveAction {
    private int start;
    private int end;
    private Double[] weights;
    public WeighAnimalAction(Double[] weights, int start, int end) {
        this.start = start;
        this.end = end;
        this.weights = weights;
    }
    
    protected void compute() {
    if(end - start <= 3) // each zoo keeper can only weigh 3 an hour
        // start weighing
        for(int i=start; i<end; i++) {
            weights[i] = (double)new Random().nextInt(100); // random animal weight
            System.out.println("Animal Weighed: "+i);
        }
    else {
        int middle = start+((end-start)/2);
        System.out.println("[start="+start+",middle="+middle+",end="+end+"]");
        invokeAll(
            new WeighAnimalAction(weights,start,middle),
            new WeighAnimalAction(weights,middle,end));
        } // this is the recursive part
    }
}

public static void main(String[] args) {
    Double[] weights = new Double[10];
    ForkJoinTask<?> task = new WeighAnimalAction(weights,0,weights.length);
    ForkJoinPool pool pool = new ForkJoinPool();
    pool.invoke(task);
    
    // Print results
    System.out.print("Weights: ");
    Arrays.asList(weights).stream().forEach(
    d -> System.out.print(d.intValue()+" "));
}
```
```
[start=0,middle=5,end=10]
[start=0,middle=2,end=5]
Animal Weighed: 0
Animal Weighed: 2
[start=5,middle=7,end=10]
Animal Weighed: 1
Animal Weighed: 3
Animal Weighed: 5
Animal Weighed: 6
Animal Weighed: 7
Animal Weighed: 8
Animal Weighed: 9
Animal Weighed: 4
Weights: 94 73 8 92 75 63 76 60 73 3
```
Some fork/ join methods to be aware of:

The ``invokeAll()`` method taks two instances of the fork/join class and does not return a result.

The ``fork()`` method causes a new task to be submitted to the pool and is similar to the thread executor ``submit()`` method.

The ``join()`` method is called after the ``frok()`` method and causes the current thread to wait for the results of a subtask.

## Threading problems
**Deadlock** occurs when tow or more threads are blocked forever, each waiting on the other. There are some common strategies to avoid deadlocks such as for threads to order their resource requests. There are also some advanced techniques to try to detect adn resolve them in real time but these are very difficult to implement and have limited success generally.

**Starvation** is whena  single thread is perpetually denied access to a shared resource or lock.

**Livelock** is when two or more threads are conceptually blocked forever although they are still active and trying to complete their task. Its often the result of two threads trying to resolve a deadlock. Its often very difficult to detect.


