---
layout: post
title:  "Thread - Thread Priority"
subtitle: 
description:
date:   2019-10-09 00:45:22 +0900
comments: true
image: /assets/img/canyon.jpg
optimized_image: /assets/img/canyon.jpg
category: java
tags: Java
author: tlonist
---

#### Thread Priority

- You can set the priorities of existing threads by using **setPriority** method. Below is the actual Java code for setPriority method.

```java
   /**
     * Changes the priority of this thread.
     * Otherwise, the priority of this thread is set to the smaller of
     * the specified <code>newPriority</code> and the maximum permitted
     * priority of the thread's thread group.
     *
     * @param newPriority priority to set this thread to
     * @exception  IllegalArgumentException  If the priority is not in the
     *               range <code>MIN_PRIORITY</code> to
     *               <code>MAX_PRIORITY</code>.
     * @exception  SecurityException  if the current thread cannot modify
     *               this thread.
     * @see        #getPriority
     * @see        #checkAccess()
     * @see        #getThreadGroup()
     * @see        #MAX_PRIORITY
     * @see        #MIN_PRIORITY
     * @see        ThreadGroup#getMaxPriority()
     */

  public final void setPriority(int newPriority) {
        ThreadGroup g;
        checkAccess();
        if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
            throw new IllegalArgumentException();
        }
        if((g = getThreadGroup()) != null) {
            if (newPriority > g.getMaxPriority()) {
                newPriority = g.getMaxPriority();
            }
            setPriority0(priority = newPriority);
        }
    }
```

Here is a concrete example using setPriority method. The default priority for any newly created thread is 5. The main below is expected to finish t2 "=" prior to "&" since t2's priority is set to 7(>5)


```java
public class Thread4 {
    public static void main(String[] args) {
        ThreadPr1 t1 = new ThreadPr1();
        ThreadPr2 t2 = new ThreadPr2();

        t2.setPriority(7);
        System.out.println("Priority of t1 : "+ t1.getPriority());
        System.out.println("Priority of t2 : "+ t2.getPriority());

        t1.start();
        t2.start();
    }
}

public class ThreadPr1 extends Thread {
    @Override
    public void run() {
        for (int i=0; i<400; i++){
            System.out.print("&");
            for(int j = 0; j<10000000; j++);
        }
    }
}

public class ThreadPr2 extends Thread {
    @Override
    public void run() {
        for (int i=0; i<400; i++){
            System.out.print("=");
            for(int j = 0; j<10000000; j++);
        }
    }
}
```

**Console Output**
```console
Priority of t1 : 5
Priority of t2 : 7
&=&=&&&&&&&==================&&&&&&&&&&&&&&&&==&&&
==================================&&&&&&&&&&&&&&&&
&&&&&=====================================&&&&&===
======&&&&&&&&===========&&&&&&&&&&&&&&&&&&&&&&&&&
&&&&&&&&&&&&&&=========&&&&&======================
===&&&===========&&&&&&&&&&&&&&&&&&&&&==&&&&&=====
============&&&&&&&&&&&&&&&&=====&&&&&&&&&&&&&&&&&
&&&&&&&&==&&&&&&&&&&&&&&&=========&&&&&&&&========
=================&&&&&&&&&&============&&&========
```

- ***???***
- It didn't work as I'd expected. This may be due to the fact that my MacBook uses multi core CPU, making each thread independent of each other. Hmm.. 
