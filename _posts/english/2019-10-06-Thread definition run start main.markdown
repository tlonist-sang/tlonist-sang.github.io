---
layout: post
title:  "Thread - Definition, run()&start(), main"
subtitle: 
description:
date: 2019-10-06 00:45:22 +0900
comments: true
image: /assets/img/alaska.jpg
optimized_image: /assets/img/alaska.jpg
category: java
tags: Java
author: tlonist
---

## Thread

> #### Definitions : Process and Thread

**Process**  is simply a **'program in execution'**. When a program is executed, OS allocates appropriate resources(memory) to the program, making it a process.

**Thread** is an actual executing module functioning within a process.

***Process** = **Threads** + Resources (Data + Memory)*

- Threads share the same address space and therefore can share both data and code
- Context switching between threads is usually less expensive than between processes
- Cost of thread intercommunication is relatively low that that of process intercommunication
Threads allow different tasks to be performed concurrently.


> #### Single & Multi Thread

CPU core performs **one task at a time**; hence, the number of concurrently processed task is equal to the number of CPUs. Because the former is almost always greater than the latter, CPU works by switching from one thread to the other at an amazingly fast speed.
![image](https://www.backblaze.com/blog/wp-content/uploads/2017/08/diagram-threads.png)
#####Is Multi-Threading always better?
- No. If threads use same resources to perform a task, there is a computation overhead of **switching between contexts.**
- Multi threading is especially effective when threads use different resources. For example, user I/O task and file transfer via network can be benefited from using multi threading. 


> #### Creating a new Thread 

**Creating a thread** is not difficult. You can either create it by extending Thread class or implementing a Runnable interface. Using Runnable interface is preferred because, once inheriting from Thread class the inheriting class cannot inherit other classes.

```java
public class Thread1 {
    public static void main(String[] args) {
        ThreadEx t1 = new ThreadEx(); //Thread creation by inheriting Thread class
    
        Runnable r = new ThreadRn(); //Thread creation by implementing Runnable interface
        Thread t2 = new Thread(r);

        //Thread t2 = new Thread(new ThreadRn()) also works
        
        t1.start();
        t2.start();
    }
}

public class ThreadEx extends Thread {
    public void run(){
        System.out.println("Hello, I'm thread 1!");
    }
}

public class ThreadRn implements Runnable{
    @Override
    public void run() {
        System.out.println("Hello, I'm thread 2!");
    }
}
```
**Console Output**
``` shell 
Hello, I'm thread 1!
Hello, I'm thread 2!
```


> #### start( ) , run( ), Main Thread 

The above code called **start()**, not **run()**. Why? 
- **start()** method creates a **call stack** necessary for a thread to execute its task. 
- Once a call stack is made, **run()** is loaded onto it.
- Whenever a new thread is created or deleted, a call stack is also created or deleted. 
- Below example demonstrates the difference between calling a start() method and calling a run() method.

```java
public class Thread2 {
    public static void main(String[] args) {
        ThreadRun t1 = new ThreadRun();
        t1.start(); //Thread called start, 
    }
}

public class ThreadRun extends Thread {
    public void run(){
        throwException();
    }
    private void throwException(){
        try {
            throw new Exception();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
**Console Output**
```console
java.lang.Exception
	at Thread.Thread2.ThreadRun.throwException(ThreadRun.java:10)
	at Thread.Thread2.ThreadRun.run(ThreadRun.java:6)
```
- Note that the error was NOT thrown in **main**. This is because the error was thrown in a different call stack, one that is claimed by 'start()' in the main method. 


```java
public class Thread2 {
    public static void main(String[] args) {
        ThreadRun t1 = new ThreadRun();
        t1.run(); //Thread is run from the Main stack, not from a new stack
    }
}
```

**Console Output**
```console
java.lang.Exception
	at Thread.Thread2.ThreadRun.throwException(ThreadRun.java:10)
	at Thread.Thread2.ThreadRun.run(ThreadRun.java:6)
	at Thread.Thread2.Thread2.main(Thread2.java:6)
```
As can be expected, the error stack includes **main** this time. 

