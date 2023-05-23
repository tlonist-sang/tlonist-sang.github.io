---
layout: post
title:  "Synchronous VS Asynchronous (1/2)"
subtitle: "Is sync == blocking and async == non-blocking ? (1/2)"
description:
date:   2020-01-15 00:45:22 +0900
comments: true
image: /assets/img/mountain_bridge.jpeg
optimized_image: /assets/img/mountain_bridge.jpeg
category: javascript
tags: Java, JavaScript
author: tlonist
---

The goal of this post is to answer some ambush questions by friends of mine or a novice programmer asking 'Yo, so what's the difference between synchronous and asynchronous models? what about blocking and non-blocking?' I am VERY TIRED of not being able to answer clearly at the spot. So here it goes!

- Synchronous : 
---------------

Suppose you have to watch two movies and write essays on them. If you decide to be honest and not look up for sparknotes or wikipedia, you will have to watch one movie, and write an essay about it. Once you are finished with one, you would have to watch another movie and after done watching you will be able to write another essay. In this case you were doing a **synchonous** job, because you had to wait until the movie is over to write an essay about it. 
[![img]({{ "/assets/img/sync1.png"|absolute_url}})]({{ "/assets/img/sync1.png"|absolute_url}})

- Asynchronous : 
----------------

In the middle of watching the movie, you realized that you forgot to feed your dog. So you fill his bowl with pet cereals, and while he is eating you go back to continue watch the movie. In this situation, you were doing two jobs - watching a movie and feeding your dog - **asynchronously**.
[![img]({{ "/assets/img/sync2.png"|absolute_url}})]({{ "/assets/img/sync2.png"|absolute_url}})

>what distinguishes synchronous and asynchronous is **whether the calling function cares for completion of called function's job**. 

If the calling function waits for return of called function's job, or cares about the completion of called function's job (even if it gets return right after calling) by checking continuously, it is **synchronous**. If the calling function does NOT care about the complemetion of called function's job, it is **asynchronous**; you will continue watching the movie once you've filled his bowl - you won't sit down and wait to see he finishes eating. 

Then let's go for actual code examples for sync and async models. In fact, this is already covered in the postings related to threads in my blog. First off, synchronous model in Java.

### Java
```java
public class Test {
    public static void main(String[] args) {
        printA();
        printB();
        printC();
    }
    public static void printA(){
        for(int i=0; i<100; i++)
            System.out.print("A");
    }
    public static void printB(){
        for(int i=0; i<100; i++)
            System.out.print("B");
    }
    public static void printC(){
        for(int i=0; i<100; i++)
            System.out.print("C");
    }
}
```

Output
```console
AAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBBBBBBBBBBBBBBBBBBBCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
```

This is just it, very plain and simple. When invoking main method above, printA, printB, printC are called in order. Note that printB is not called until printA is **finished**, and printC is not called until printB is finished. Likewise the methods above look out for the **completion of the executed prior to itself**, behaving **synchronously**.

Let's now look at the asynchronous model in Java. If two methods are executed asynchronously, they should NOT care whether the other one is finished or not, and just keep on executing its own. This can be accomplished by making two **separate threads to execute**, like below.

```java
    public static void main(String[] args) {
        Thread horizontal = new Thread(new Thread1());
        Thread vertical = new Thread(new Thread2());

        //executed in order 
        horizontal.start();
        vertical.start();
    }

    public class Thread1 implements Runnable {
        @Override
        public void run() {
        for(int i=0; i<100; i++)
            System.out.print("-");
        }
    }

    public class Thread1 implements Runnable {
        @Override
        public void run() {
        for(int i=0; i<100; i++)
            System.out.print("|");
        }
    }
```

Output
```console
------------||||---------------||||||||||||||||||||||||||||||||-----------||||||||||||||||||||||||||||||||||--------------------------------------------------------------||||||||||||||||||||||||||||||
```

Though it was executed in order, just like the methods printA-C had been. But this time, the output is not in the order of method calling; the horizontal start() and vertical start() do not care about whether the other one is finished, but keep on doing what they have to do - just like your dog won't care whether you are wathcing a movie and you don't care whether your dog is finished eating. 

()
In Java, the asynchronous model can be easily achieved by making multiple threads run concurrently.
[![img]({{ "/assets/img/sync3.png"|absolute_url}})]({{ "/assets/img/sync3.png"|absolute_url}})

Of course, these multiple threads can be made run synchronously. 
[![img]({{ "/assets/img/sync4.png"|absolute_url}})]({{ "/assets/img/sync4.png"|absolute_url}})

Even more, a single thread in Java can be made synchronously and asynchronously as well. But I think **JavaScript**, which is natively single-thread, can explain better. So here it goes.

### JavaScript
Unlike Java, JavaScript uses only single thread. Synchronous model is exactly the same with that of Java.
```javascript
const printA = ()=>{
    printB();
    console.log("A");
}
const printB = ()=>{
    printC();
    console.log("B");
}
const printC = ()=>{
    console.log("C");
}
printA();
```

```console
C
B
A
```

Async example will be like below.
```javascript
console.log('1')

setTimeout(()=>{
  console.log('2')
}, 2000)

console.log('3')
```
```console
1
3
2
```
As can be noted from the console output, you can see that instead of waiting for 2 seconds from **setTimeOut**, the code went on printing 3 out. Hence 1->3->2.

The major difference between Java and JavaScript codes introduced above stems from the number of threads each uses for implementing synchronous and asynchronous models. Unlike Java, JavaScript uses only single thread, hence the diagram for single thread sync & async model will be..

[![img]({{ "/assets/img/syncsingle2.png"|absolute_url}})]({{ "/assets/img/syncsingle1.png"|absolute_url}})
In synchronous model, each task(job, function, method... wtv) is performed on completion of its previous task.

[![img]({{ "/assets/img/syncsingle1.png"|absolute_url}})]({{ "/assets/img/syncsingle2.png"|absolute_url}})
Whereas in asynchronous model, from single thread, the tasks are performed by bit by bit, switching contexts, until they are fully executed with return.

Summing up, in this posting I covered
- What it means to be synchronous and asynchronous in programming
- Examples of synchronous and asynchronous models in Java with code 
- Examples of synchronous and asynchronous models in JavaScript code

In the following posting, I will cover blocking and non-blocking natures in programming. See you!


references
- [https://medium.com/swift-india/concurrency-parallelism-threads-processes-async-and-sync-related-39fd951bc61d](https://medium.com/swift-india/concurrency-parallelism-threads-processes-async-and-sync-related-39fd951bc61d)
- [https://www.geeksforgeeks.org/asynchronous-synchronous-callbacks-java/](https://www.geeksforgeeks.org/asynchronous-synchronous-callbacks-java/)
- [https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/#about](https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/#about)
- [https://djkeh.github.io/articles/Boost-application-performance-using-asynchronous-IO-kor/](https://djkeh.github.io/articles/Boost-application-performance-using-asynchronous-IO-kor/)