---
layout: post
title:  "Thread - Daemon Thread"
subtitle: 
description:
date:   2019-10-09 00:45:22 +0900
comments: true
image: /assets/img/new-zealand.jpg
optimized_image: /assets/img/new-zealand.jpg
category: java
tags: Java
author: tlonist
---

#### Daemon Thread
**Daemon thread** is a subsidiary thread that helps other non-daemon threads' tasks. You can easily think of Java's garbage collection, WordProcessor's auto-save, or auto-screen reloading functionalities.

- It terminates when all other non-daemon threads terminate.
- It can not prevent the JVM from exiting when all the user threads finish their execution.
- If JVM finds running daemon thread, it terminates the thread and after that shutdown itself. JVM does not care whether Daemon thread is running or not.
- It is an utmost low priority thread.

```java
public class ThreadDaemon implements Runnable{
    static boolean autoSave = false;

    public static void main(String[] args) {
        Thread t = new Thread(new ThreadDaemon());
        t.setDaemon(true);
        t.start();

        for(int i=0 ; i<10; i++){
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {}
            System.out.println(i);
            if(i==5){
                autoSave = true;
            }
        }
        System.out.println("Exiting application..");
    }
    @Override
    public void run() {
        while(true){
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {}
            if(autoSave){
                autoSave();
            }
        }
    }
    public void autoSave(){
        System.out.println("Automatic Backup!");
    }
}
```
**Console Output**
```console
0
1
2
3
4
5
Automatic Backup!
6
Automatic Backup!
7
Automatic Backup!
8
Automatic Backup!
9
Exiting application..
```
- Once the variable 'autoSave' is set true, the autoSave() function is called for every second(Thread.sleep(1000)). 

- One thing to note, **setDaemon()** must be called before **start()**. Since a daemon thread is subordinate to the working non-daemon thread, it has to be configured in advance; otherwise **IllegalThreadStateException** occurs.