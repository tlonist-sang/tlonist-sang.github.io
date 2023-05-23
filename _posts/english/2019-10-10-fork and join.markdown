---
layout: post
title:  "fork and join"
subtitle:
description:
date:  2019-10-10 00:45:22 +0900
comments: true
image: /assets/img/fork.png/
optimized_image: /assets/img/fork.png
category: java
tags: Java
author: tlonist
---

#### Fork & Join

Multithread programming with **fork & join** is 'very' fun.
From JDK 1.7 onward, a framework called 'fork & join' is added. This framework makes it possible to split a task into many units of tasks, each of which is processed concurrently by different threads.


>RecursiveAction - task without any return value
>RecursiveTask - task with return value

What you need to do is to implement **compute()** method by inheriting certain classes as

```java
public class SumJob extends RecursiveTask<Long> { 
    long start, end;
    SumJob(long start, long end){
        this.start = start;
        this.end = end;
    }
    @Override
    protected Long compute() {
        //Task is divided 
        long size = end - start + 1;
        if(size <= 10)
            return sum();

        long half = (start + end)/2;

        SumJob leftJob = new SumJob(start, half);
        SumJob rightJob = new SumJob(half+1, end);

        leftJob.fork(); //fork is asynchronous
        return rightJob.compute() + leftJob.join(); //join is synchronous

    }
    public long sum(){
        long ret = 0l;
        for(long i = start; i<=end; i++){
            ret += i;
        }
        return ret;
    }
}

public class ForkMain {
    static final ForkJoinPool forkJoinPool = new ForkJoinPool();
    public static void main(String[] args) {
        long startTime;
        long start= 1;
        long end = 1000000;
        long result = 0;

        SumJob sumJob = new SumJob(start, end);

        startTime = System.currentTimeMillis();
        result = forkJoinPool.invoke(sumJob);
        System.out.println(result + ", Time taken is " + (System.currentTimeMillis()-startTime));

        result = 0;
        startTime = System.currentTimeMillis();
        for(long i = start; i<=end; i++){
            result += i;
        }
        System.out.println(result + ", Time taken is " + (System.currentTimeMillis()-startTime));
    }
}
```

**Console Output**
```console
500000500000, Time taken is 44
500000500000, Time taken is 4
```

Time taken for multithread operation is much longer than that of simple single thread. Hence you should always investigate if using multi thread programming will actually be beneficial for certain operations.