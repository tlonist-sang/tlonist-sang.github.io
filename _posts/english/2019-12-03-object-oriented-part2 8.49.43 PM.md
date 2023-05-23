---
layout: post
title:  "Object Oriented Programming - 2"
subtitle:
description:
date:   2019-12-03 00:45:22 +0900
comments: True
image: /assets/img/nevada.jpg
optimized_image: /assets/img/nevada.jpg
author: tlonist
category: java
tags: Java
---

(Continuing from the previous post...)

So I covered the basics to enter the world of OOP, and got to know that OOP is a way of programming that deconstructs the world with objects and interactions among objects. I covered 
1) How to instantiate objects
2) What comprises a class (variables and methods)
3) JVM memory structure
4) Various methods for initializing variables

In this posting, I will try to get hold of mainstream OOP features of Java language, namely encapsulation, polymorphism, inheritance, and abstraction. I'm not much of a novice to learn these concepts all anew, but I'd like to solidify my understanding and correct any misunderstanding if I had any.

### 2. Inheritance

Inheritance in OOP is to write a new class by re-using an existing class. 

 ```java
 class Child extends Parent{
     //Child class inheriting the Parent class.
 }
 ```

 - Constructors and initializing blocks are NOT inherited. Only member variables are inherited
 - (Naturally) The number of member variables in the child class is always greater or equal to that of the parent class.
 
 The concept is well-known and quite easy to understand. So I will only go over some aspects that I've not been so familiar with. 

 #### 2-1. Single Inheritance
 ```java
  class Child extends Mom, Dad //error! cannot have two ancestors at once.
  class SomeClass extends Object //every class is descended from the Object class.
 ```
 It is noteworthy to think why this is not allowed in Java.
 1. If two parents have member variables with same name, it is impossible for the child to distinguish, unless they are static. 
 2. Continuing from 1, if one of the parents has to change the variable, all the classes that make use of the variable or the method has to change the code.

There is a roundabout way like below.

```java
 class Horse {
     void run();
 }
 
 class Bird {
     void sing();
 }

 class SingingHorse extends Horse{
     Bird bird = new Bird();
     void sing(){
         bird.sing(); // Let the freakin' bird inside sing.
     }
 }
```
It seems that the horse is singing, but in fact it is the bird within the SingingHorse that's singing.
By doing so, when the Bird class is changed, the SingingHorse class can also be changed, as if it is inherited from the Bird class itself.

#### 2-2. Overriding
```java
 class 2DPoint {
     int x;
     int y;
     String getLocation(){
         return x + "," + y;
     }
 }

 class 3DPoint {
     int z;
     @Override
     String getLocation(){ //overriding
         return x + "," + y + "," + z;
     }
}
```
Overriding is changing the method inherited from the parent class. In order to override a method, three things method name, parameter, and the return type must remain the same from those of parent class.

- Acess modifier(public, protected, default, private) cannot have a narrower range that that of the parent. 
- Overriden methods cannot have more exceptions than the parent method. 
- Static method is not overriden. They are just bound to the declaring class. (It is possible that the child and parent class have the same static methods. The methods are separate, individual methods and have nothing to do with each other)

This should not be confused with **overloading**, which is defining a new method.

#### 2-3. super

```java
public class TestSuper {
    public static void main(String[] args) {
        Child c = new Child();
        c.method1();
        System.out.println(c.method2());
    }
}
public class Parent {
    int x = 10;
    public String method2(){
        return "hello";
    }
}
public class Child extends Parent {
    int x = 10;
    public void method1(){
        System.out.println(this.x);
        System.out.println(super.x);
    }

    public String method2(){
        return super.method2() + " world!";
    }
}
```

```console
10
hello world!
```
As can be seen, super is a variable that is used by the child class to refer to the member variables or the methods in the parent class. **super()** is a constructor version of **super**. 


### 3. Polymorphism

As the name suggests, polymorphism means 'ability to have many forms'; and in Java, this means that one reference variable can reference other types of objects. To be more specific, parent class objects can reference child class instances. I will use Clash Royale, which is the game I've been playing for quite a long time, for examplification. 
[![img]({{ "/assets/img/cr1.png"|absolute_url}})]({{ "/assets/img/cr1.png"|absolute_url}})


```java
class RoyalObject{
    String name;
}

class Unit extends RoyaleObject{
    int hitpoint;
    int weight;
    int size;
    int damage;
    int level;
    Rarity rarity;  //enum value    
}

public static void main(String[] args){
    RoyalObject royalObject = new Unit();   //this is possible!    
    Unit unit = new Unit();                 //What might be the difference between the upper line and this?
}
```

[![img]({{ "/assets/img/oop2-1.png"|absolute_url}})]({{ "/assets/img/oop2-1.png"|absolute_url}})

The above diagram shows the difference between using parent class reference variable to refer child class instance.
On surface, there doesn't seem to be any merit in implementing polymorphism, because the restriction on parent class is still the same. Then why do we bother do this? 

#### **3-1. Type Casting**

Type can be cast from parent class instance to child class instance, and vice versa. 

```java
public static void main(String[] args){
    RoyalObject royalObject = null;
    Unit unit1 = new Unit();   
    Unit unit2 = null;

    royalObject = unit1;
    //unit2 = royalObject;        //error!!
    unit2 = (unit)royalObject;  //correctly type cast
}
```

- Bear in mind that royalObject is just a **reference variable**, which is an object holding the reference address of actual instance.
- In this light, although **royalObject** has restrictions on accessing unit type variables, the unit2 reference variable will have **full** access to the unit instance created by **new Unit()**.
- Type casting, in essense, controls the number of variables a reference variable can access.

#### **3-2. instanceof**

Knowing the actual type of instance a reference variable refers is quite important. 

```java
public static void main(String[] args){
    RoyalObject royalObject = new RoyalObject;
    Unit unit = null;

    //...
    //some 3000 lines of code
    //...

    unit = (unit) royalObject;   //I'll typecast it
    
    //...
    //some 3000 lines of code
    //...

    //error!!
}
```
- The error like above **may break out** after long long lines of codes in the middle when you forgot how royalObject was defined in the first place. (But IDEs have become so intelligent that this almost never takes place.. I'm just saying for the sake of explanation)
- In such case, let's use **instanceof**. Very simple

```java
void doSomething(RoyalObject r){
    if(r instanceof Unit){
        System.out.println(r.getHitPoint()); //does this work? NO!!
    }

    if(r instanceof Unit){
        System.out.println((Unit)r.getHitPoint()); //Proper type-casting is required.
    }
}
```

#### **3-3. Polymorphism in method parameter**

**UNITS - knigt, giant, and hog rider!**

[![img]({{ "/assets/img/cr4.png"|absolute_url}})]({{ "/assets/img/cr2.png"|absolute_url}})

```java
class Unit extends RoyaleObject{
    int hitpoint;
    int weight;
    int size;
    int damage;
    int level;
    Rarity rarity;  //enum value    
}

class Knight extends Unit{}
class Giant extends Unit{}
class HogRider extends Unit{}
```
And we have battle field, classic challenge, in which all the units' level is adjusted to 9.

```java
class ClassicBattleField extends BattleField(){
    private static int level = 9;

    public Knight initializeHpByLevel(Knight knight){
        int knightHp = calculateHP(knight.getHitPoint(), level);
        knight.setHitPoint(knightHp);
        return knight;
    }
}
```

and because we have giant and hog rider as well, 
```java
public Giant initializeHpByLevel(Giant giant){
    int giantHp = calculateHP(giant.getHitPoint(), level);
    giant.setHitPoint(giantHp);
    return giant;
}

public HogRider initializeHpByLevel(HogRider hogRider){
    int hogRiderHp = calculateHP(hogRider.getHitPoint(), level);
    hogRider.setHitPoint(hogRiderHp);
    return hogRider;
}
```

- This is **utterly** very tedious. But this is where polymorphism butts in.

```java
public Unit initializeHpByLevel(Unit unit){
    int unitHp = calculateHP(unit.getHitPoint(), level);
    unit.setHitPoint(unitHp);
    return unit;
}
```
- This has become amazingly easy to code and manage. You can set HP for units using super() method.
```java
class Knight extends Unit{ super(1000); } //initializes HP of Knight unit
class Giant extends Unit{ super(1600); }
class HogRider extends Unit{ super(1100); }
```
[![img]({{ "/assets/img/cr6.png"|absolute_url}})]({{ "/assets/img/cr6.png"|absolute_url}})
