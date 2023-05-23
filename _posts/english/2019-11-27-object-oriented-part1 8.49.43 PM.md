---
date: 2019-11-27 00:45:22 +0900
layout: post
title:  "Object oriented programming - 1"
subtitle: basic knowledge
description: ingredients for oop!
image: /assets/img/himalayan.jpg
optimized_image: /assets/img/himalayan.jpg
category: java
tags: Java
author: tlonist
---

What is an object-oriented programming? and why is it so important?
Do I really understand what it is?


#### Object-Oriented Language - Abstract
> Simply put, it is a viewpoint that the world is made up of objects whose interactions make up all the events happening in the world.

- Major properties of Object-oriented languages
    - high re-usability
    - easy to manage and refactor
    - high integrity
Anyways, so it's **GOOD!**
<br>

- I'll talk about four major features that Java has as an object-oriented language, namely **'Abstraction', 'Polymorphism', 'Inheritance', 'Encapsulation'.**
<br>


- Before going further, let us go over the very basics first.

#### 1. Basics

##### 1-1. Object Instantiation

Though I've been using class and object without pondering their definitions, they were quite hard to define when I had to. Here they are from a book that I'm referencing.

**A class is a blueprint** that **defines an object**
**An object** is **something that exists**

Making an object from a class is called **instantiation**.

```java
Fruit a;          //declaring a reference variable with type Fruit
a = new Fruit();  //After making a fruit instance, the address of fruit instance is saved in a
```

[![img]({{ "/assets/img/oop1.png"|absolute_url}})]({{ "/assets/img/oop1.png"|absolute_url}})

- One thing to note is that the instantiation and declaration is a separate thing; **and I think this plays a crucial role in understanding what will come next with object-oriented programming.** Instance can only be manipulated by its referece variable, and the type of reference variable **must match** with that of instance type. 

##### 1-2. Object Array

```java
Fruit[] fArr = new Fruit[3]; // Fruit type reference array with length 3
```
Remember, the instantiation did not take place yet. You may want to do something like below for actual creation of objects.

```java
for(int i=0 ; i<3; i++){
    fArr[i] = new Fruit();
}
```

##### 1-3. Variable and Method

```java
class SCSA{
    ////
    int student_number;         //instance variable
    static int teacher_number;  //class variable
    ////    class area

    void study(){
        ////
        int period = 6;         //local variable
        //// method area
    }
}
```
| kinds             | position                                 | time of creation                  |
|-------------------|------------------------------------------|-----------------------------------|
| class variable    | class area                               | when class is loaded to memory    |
| instance variable | class area                               | when instance is made             |
| local variable    | method, constructor, initializing blocks | when variable declaration is made |


##### 1-4. JVM memory structure

```java
public class CallStackTest {
    public static void main(String[] args) {
        methodOne();
    }
    static void methodOne(){
        methodTwo();
    }
    static void methodTwo(){
        System.out.println("methodTwo!");
    }
}
```

- Method area
    : When a class is used for a program, JVM reads its class file (*.class) and stores its class data in method area. 
- Heap
    : place where instances are created. All the instances created during the execution of a program is stored here.
- Call stack
    : offers memory space for operation of methods. When a method is called, a memory is allocated to a call stack for the method to operate, which then is used for computations and storing of results etc. When the method finishes, it returns the memory space reserved for computation.

Below diagrams show the flow of the **CallStackTest**

[![img]({{ "/assets/img/oop2.png"|absolute_url}})]({{ "/assets/img/oop2.png"|absolute_url}})

##### 1-5. Initializing variables

```java
Class Test{
    int x;
    int y = x;  //okay

    void test1(){
        int i;
        int j = i;  //error!
    }
}
```
- Member variables does not necessitate initialization; whereas local variables must be initialized. 
- There are several ways to initialize a member variable.
    - __explicit initialization__
    ```java
    class Test{
        int x = 3; //explicitly initializing a member variable! done.
    }
    ```
    - **constructor**
    ```java
    class Test{
        int number;
        String name;

        Test(){} //constructor
        Test(int number, String name){
            this.number = number;
            this.name = name;
        }
    }
    ```

    - **initialization block**
    ```java
    class Test{
        static { /*  class initializing block  */}
        {/*  instance initializing block  */}
    }
    ```
    As can be noted, class initializing block is executed only once when a class is first loaded to the memory; instance initializing block is executed every time when an instance is instantiated. 
    It is also noteworthy that **initializing blocks always precede constructors**.

    ```java 
    class Test {
        int id;
        int element;
        {
            id = 1;
            element = 2;
        }
        public Test(){
            id = 98;
            element = 99;
        }
    }
    ```
    For situation above, creating Test test = new Test(); will render test.id to be 98 and element to be 99.

    <br>

    In the following postings, I'll cover the real OOP topics.