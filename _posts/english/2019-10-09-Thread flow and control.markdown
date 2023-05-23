---
layout: post
title:  "Thread flow and control"
subtitle:
description:
date:   2019-10-09 00:45:22 +0900
comments: true
image: /assets/img/grapes.jpg
optimized_image: /assets/img/grapes.jpg
category: java
tags: Java
author: tlonist
---

#### Thread flow and control


 ![img]({{ "/assets/img/Threadflow.png"|absolute_url}})

- **sleep(long millis)** : stops a thread for a given amount of time (static method)
```java
public class ThreadSleepMain {
    public static void main(String[] args) {
        tsleep1 t1 = new tsleep1(); 
        tsleep2 t2 = new tsleep2();

        //t1 and t2 are simple threads that prints - and |, respectively

        t1.start();
        t2.start();

        try {
            t1.sleep(2000);
        } catch (InterruptedException e) {}
        System.out.println("finish!");
    }
}
```
**Console**
---------||||||||||||||||||||||||||||||||||||||----------||||||||||||||||||||||||||||||||||||||||||||||------------------------|||||||||||---||||----------|||||||||||||||-----------------------|||||||||||-------------------------------------------------------------------------------------------|||||||||||||||----------------|||||||||||||----------------------------------------------------------------------------|||||||||||||---||||||||||-----------------------------------|||||||||||||||||||||||||||||||||||t1 terminated
|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||t2 terminated
finish!

The above result is different from time to time; sometimes t2 finishes first. Note that **sleep** applies for the current working thread. To properly apply sleep method to t1, the code has to be inserted into t1 class directly.

- **interrupt()** : cancels thread's task

```java
public class ThreadInterrupt {
    public static void main(String[] args) {
        Tinterrupt t1 = new Tinterrupt();
        t1.start();
        String input = JOptionPane.showInputDialog("Enter any value");
        System.out.println("You entered : "+input + ".");
        t1.interrupt();
        System.out.println("is Interrupted "+t1.isInterrupted());
    }
}

public class Tinterrupt extends Thread {
    @Override
    public void run() {
        int i = 10;
        while (i!=0 && !isInterrupted()){
            System.out.println(i--);
            // try {
            //     Thread.sleep(1000);
            //     Thread.interrupted();
            // } catch (InterruptedException e) {
            //     e.printStackTrace();
            // }
            for(long x= 0; x<2500000000L; x++);
        }
        System.out.println("Count finished.");
    }
}
```
**Console Output**
```console
10
9
8
7
6
5
4
You entered : 1.
is Interrupted true
Count finished.
```
Interesting thing happens when you put the thread to sleep(). The isInterrupted sometimes shows false() and even after inputting some value the count goes on. This is because **sleep()** causes InterruptedException. Adding interrupted() inside the try-catch block sets the state back to the original one.


- **suspend(), resume(), stop()** (Not covering)
- **yield()** (Not covering)
- **join()** : waits for other thread's task is done.

```java
public class ThreadJoinMain {
    static long startTime = 0;

    public static void main(String[] args) {
        Threadj1 t1 = new Threadj1();
        Threadj2 t2 = new Threadj2();

        //t1 and t2 prints - and | 200 times, respectively.

        t1.start();
        t2.start();

        startTime = System.currentTimeMillis();
//        try {
//            t1.join();
//            t2.join();
//        } catch (InterruptedException e) {
//            e.printStackTrace();
//        }
        System.out.println("Time taken: "+ (System.currentTimeMillis()-startTime));
    }
}
```

**Console Output (commented)**
```console
Time taken: 0
-------||-------------------|||||||||||||---------||||||||||------------------------------------------------------------------------------|||||||||---||||||-------------||||||||||||-----------||||------|||||||||||||||||||||||||------||||||||----------|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||--------------------------------------

```

**Console Output (comment disabled)**
```console
---------------------------------------||||||||||||||-----|||||------||||||||||||||||||||||||-------|||||||||||||||||||||||--|||||||------|||||||||------------------------||||||||||||||||||||||||||||||||||||||||||||------------||||||||||||-------|||||||||||||------||||||||||||||||||||---------|||||||||||||||||||||||------||||--------------------------||---------------------------------------------
Time taken: 4

```
As can be noted, when **join()** is called from the main stack, threads t1 and t2 intervene and only after they are finished the rest of main thread is working.
One more thing to note is that **join()** is different from **sleep()** in that it is not a static method; it is specific to the object calling the method.