---
layout: post
title:  "Brief discussion on Garbage Collector"
subtitle: "brief mechanism underneath"
description:
date:   2020-06-28 00:00:00 +0900
tags: java
comments: True
image: /assets/img/garbage.jpeg
optimized_image: /assets/img/garbage.jpeg
author: tlonist
category: Java
---

### Prelude
In this posting, I will write about
- What a garbage collector is
- Brief ways of tuning GC 

This article is a somewhat literal translation of an article ([link](https://d2.naver.com/helloworld/1329)), digested with my own understanding and sometimes omitting and appending details. 

### 1. What is a Garbage Collector?

- In Java, unlike C and C++, there is not a program code for declaratively freeing an allocated memory. In other words, there is not **free(something)** method that releases the previously allocated memory. 

- Sometimes a developer can do actions above by setting **null** to an object, or by calling **System.gc()**. (You should never use System.gc(), by the way). But generally, Java developers do not free an allocated memory declaratively; instead, the Garbage Collector thread does the job for you. How convenient!

- The garbage collector was deviced upon two hypothesises, called **'weak generational hypothesis'**. 
    - Most objects become unreachable almost right after being used.
    - Referencing from an older object to a younger object happens very rarely.
Hotspot VM (java virtual machine) has two distinct physical spaces to make a good use of this hypothesis: Young area and Old area.

- 'stop-the-world' is probably the most important term when it comes to GC, because the purpose of tuning GC boils down to reducing the time taken for 'stop-the-world' process. stop-the-world refers to the stopping of the whole JVM application for executing the garbage collection. 

[![img]({{ "/assets/img/gcflow.png"|absolute_url}})]({{ "/assets/img/gcflow.png"|absolute_url}})

- Young generation (minor GC): is sub-divided into two areas.
    - Eden : is where newly made objects are stored.
    - 2 Survivors : is where surviving objects go after GC-ed from Eden

    - Why are there two survivors?


- Old generation (major GC) : 
    - Major GC is executed when data is fully occupying the old generation area. 
        - Serial GC
        - Parallel GC
        - Z GC
        - Concurrent Mark & Sweep GC
        - G1(Garbage First) GC

### 2. Ways of tuning GC

#### 2-1 Serial GC 
- Serial GC mark-sweep compact algorithm. It marks which parts in memory are in use and which are not. Then it sweeps, removing objects identified during the 'mark' phase. 
- Since JVM has to keep track of object reference creation and deletion, this requires more CPU power on top of the original application. 

There are several things to consider when using this GC.
1. Total size of heap
    - Since GC occurs when generation is full, the size of usable memory is inversly proportional to the throughput of GC. (the fact that there is little usable memory indicates lots of GC operations are in need)

    [![img]({{ "/assets/img/commitedvirtual.png"|absolute_url}})]({{ "/assets/img/commitedvirtual.png"|absolute_url}})
    - -Xmx can modify the 'total' area.
    - -Xms can modify the 'committed' area.
    - Above options should be added either to the runtime environment of java (system preferences -> java) or to a specific JVM setting for an application.

    
2. Proportion of Young generation in heap memory
    [![img]({{ "/assets/img/heapproportion.png"|absolute_url}})]({{ "/assets/img/heapproportion.png"|absolute_url}})
    
    - Suppose that the heap size is fixed.
    Since minor GC occurs whenever young generation is full, increasing the size of young generation makes minor GC less frequent.
    On the other hand, if young generation takes bigger proportion from the heap, the major GC will occur more frequently, due to its smaller size.

    - The proportion can be customized by using **-XX:NewRatio **
    -XX:NewRatio=3 makes young:old = 1:3 [link](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/sizing.html)

    - Other options are
        - -XX:NewSize => sets minimum value for young generation size
        - -XX:MaxNewSize => sets maximum value for young generation size
        - -XX:SurvivorRatio => sets Eden:Survivor ratio (ex -XX:SurvivorRatio=6 makes Eden:Survivor = 6:1)
    
    - This set of instructions may be useful when tuning GC.
        - Provide maximum heap size to the JVM to start with.
        - Measure the size of young generation for optimization.
        - Maximum heap size should be smaller than system memory size.
        - When total heap size is fixed, increasing the size of young(old) sacrifices the size of old(young).
        - The size of old generation should be big enough to hold all the live objects plus 10~20% of spare space.
        - Since the allocations to young & old areas can be done parallel, if processors are added, so should increase the size of the young generation.
    
#### 2-2. Parallel GC
- The basic algorithm is the same with serial GC; parallel GC, however, as the name suggests, uses multiple threads when processing GC. Hence, it is much faster. This strategy is useful when there are enough memory and cores.    

[![img]({{ "/assets/img/parallelgc.png"|absolute_url}})]({{ "/assets/img/parallelgc.png"|absolute_url}})


#### 2-3. CMS GC (-XX:+UseConcMarkSweepGC, deprecated from JDK9, why did I bother writing this?)

[![img]({{ "/assets/img/cmsgc.png"|absolute_url}})]({{ "/assets/img/cmsgc.png"|absolute_url}})

- In the initial stage, CMS GC searches for live objects from objects closest from the class loader. This requires very short stop-the-world time.
- In concurrent mark stage, GC traces back the objects referenced by the marked objects (live) from the initial stage. The process takes place concurrently starting from this stage.
- In remark stage, GC identifies newly added or de-referenced objects.
- In concurrent sweep stage, it garbage collects unused objects. 

- CMS GC has a very short stop-the-world time. This is used when having a low latency is important. 

- Drawbacks:
    - uses more CPU and memory than other GCs.
    - compaction stage is provided by default.
        - This has to be taken cautiously. If lots of compartmentalized memories lead to compaction, it may take longer stop-the-world than other GC strategies. Hence, one needs to trace how often and long the compaction takes place.

#### 2-4. Z GC

- ZGC is a scalable low latency GC. kind of an upgraded version of CMS GC.

- It does all the expensive works concurrently, and does not stop the running threads from the application.

- This is for applications that requires 10ms or lower latency with tera-byte size heap memory.

#### instructions
[https://docs.oracle.com/javase/9/gctuning/toc.htm](https://docs.oracle.com/javase/9/gctuning/toc.htm)

[johngrib-blog](https://johngrib.github.io/wiki/java-gc-tuning/#survivor%EB%8A%94-%EC%99%9C-%EB%91%90-%EA%B0%9C%EC%9D%B8%EA%B0%80)

- Following is a helpful guide on how one should tune the GC for applications.

    - Application is lenient on stop-the-world time.
        - Let VM choose whatever GC it wants.

    - Application deals with 100MB or lower data set.
        - Use serial GC (-XX:+UseSerialGC)

    - Application runs on a single processor, and doesn't care much about the stop-the-world.
        - Use serial GC (-XX:+UseSerialGC)

    - Application prioritizes performance over anything, and is fine with 1 sec or longer stop-the-world.
        - Use parallel GC(-XX:+UseParalleGC), or let VM make its own choice.

    - Application prioritizes latency over throughput, and stop-the-world has to be less than 1 second.
        - Use G1GC (not discussed here. but refer to [link](https://www.oracle.com/technical-resources/articles/java/g1gc.html))
        - Use CMS GC (-XX:+UseConcMarkSweepGC)

    - Application prioritizes latency above anything, and uses very big heap.
        - Use ZGC (from jdk 11 on)