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

## Polling with Sleep

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
Example matching letters and numbers printing:

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
 However if we don't need the results of the traks and are finished using our thread executor there is a simpler approach:

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

