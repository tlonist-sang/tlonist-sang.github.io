---
layout: post
title:  "Object Oriented Programming - 3"
subtitle:
description:
date:   2019-12-03 00:45:22 +0900
tags: Java
comments: True
image: /assets/img/new-york.jpg
optimized_image: /assets/img/new-york.jpg
author: tlonist
category: java
---

So far I've covered Inheritance and Polymorphism. Now we have abstraction and encapsulation left. I will go on with the previous example of Clash Royale characters in explaining its concepts.

### 3. Abstraction

I think abstraction is one of the key abilities developers must have. Perhaps this is more important than ability to implement methods or using proper libraries because the stage of abstraction in any kinds of programs affects the entire product for a long time.

#### 3-1. Abstract Class
**Abstract class** is a class that contains at least one abstract method. 
Let's think about adding a new method to our usual class Unit.

```java
abstract class Unit{
    ...
    abstract void attack();     //abstract method
}
```

A new card has come out, namely a **Baby Dragon**
[![img]({{ "/assets/img/cr7bb.png"|absolute_url}})]({{ "/assets/img/cr7bb.png"|absolute_url}})

This baby dragon has a particular way of attacking, **from the sky.**.
A developer, thinking that this situation may be repeated for other cards that will be added in the future, decided to make attack() an abstract class, and let **child classes of Unit** finish the implementation.

```java
public class BabyDragon extends Unit{
    ...
    void attack(...){
        // implementing attacking for baby dragon
    }
}
```

One may ask, why do I need to add 'abstract' modifier to the attack method? just leaving it without abstract would also work, because child class can override it anytime suitable. We use **abstract** to **force** child classes to implement the abstract method, so that every child class has its own implementation of the method. Let's look into more details.

```java
public class Babarian extends Unit{
    void attck(){
        // barbarian attacking motion
    }
}

public class Prince extends Unit{
    void attack(){
        // prince attacking motion
    }
}

public static void main(String[] args){
    Unit[] army = new Unit[];
    army[0] = new Barbarian();
    army[1] = new Prince();

    for(Unit unit : army){
        unit.attck(); // each unit attacks with its own motion.
    }
}
```
<img src="https://media1.tenor.com/images/6449750a47d565b5d38ef7762becaa29/tenor.gif?itemid=7313243" width="400" height="500" />
![](https://tenor.com/view/clash-royal-attack-video-game-gif-7313243)
- From the code above, only units, the reference variables of parent class, attack. But the way of attacking will be implemeneted differently according to which object the reference variable refers to. Hence the motion of units will vary, as can be noted from the gif above.


#### 3-2. Interface

If abstract class is an unfinished blueprint, interface is a basic blueprint. It can only have abstract methods and constants as its members. 
Because it has only abstract methods and constants as its members, it is possible that a child class(interface) **inherits from multiple parents(interface)**.
<br>
One feature of hog rider unit is that it can jump rivers.
<!-- [![img]({{ "/assets/img/oop3-1.png"|absolute_url}})]({{ "/assets/img/oop3-1.png"|absolute_url}}) -->
[![img]({{ "/assets/img/hog-jump.gif"|absolute_url}})]({{ "/assets/img/hog-jump.gif"|absolute_url}})

and for explanation's sake let's say hog rider also shouts. (which is true)

```java
interface Jumpable{
    void jump();
}

interface Shoutable{
    void shout();
}

interface Hogridable extends Jumpable, Shoutable(); //only those who can shot and jump with a hog can be a hog rider!
```

Recalling from the previous posting, the hog rider class would look like this.
```java
class HogRider extends Unit implements Hogridable{ //this means "HogRider extends Unit AND HogRider implements HogRidable"
    // ... (what unit can do)
    void jump();
    void shout();
}
```

#### 3-3 Polymorphism with Interface
```java
public static void main(String[] args){
    HogRidable h = new HogRider(); //typecasting can be omitted
}

void some_method(HogRidable h){
    //...
}
```
Naturally, interface variables can be used in the method parameter, as parent class reference variables could be used in similar situations. But there is one thing different; it must pass on an instance variable of class that **implemented the interface**. Continuing from the code above, 

```java
public static void main(String[] args){
    HogRidable h = new HogRider(); //typecasting can be omitted
    some_method(h); //okay

    //HogRidable h = new HogRidable(); //Does not work from the beginning. Abstract classes can't be instantiated.
}

HogRidable some_method2(){
    HogRider hogRider = new HogRider();
    return hogRider;    //returning the implementation of HogRidable
}
```

#### 3-4 More about Interface

Recently, there has been an addition of a new card, the BATTLE HEALER! (https://clashroyale.fandom.com/wiki/Battle_Healer)
[![img]({{ "/assets/img/cr8.png"|absolute_url}})]({{ "/assets/img/cr8.png"|absolute_url}})

This is a flying unit that **heals** on every attack it does.
Before going through further discussion, let's have think about the current abstraction diagram of our cards.

[![img]({{ "/assets/img/oop3-2.png"|absolute_url}})]({{ "/assets/img/oop3-2.png"|absolute_url}})

This battle healer has following features.
- she can **jump** the river. (wings she has)
- she can heal **only ground units**

Then below follows quite naturally.
```java
interface Healable();    //it means the unit can be 'healed'
class BattleHealer extends Unit implements Jumpable, Healable{
    //...
    void heal(Healable h);
}
```

I've never thought about grouping units with ground units and flying units before, but now it seems to be necessary.
And Knight, HogRider, Giant, Battle healers are all ground units, while Baby dragon is an air unit. Writing down further specifications...

```java
class Unit extends RoyaleObject{}
class AirUnit extends Unit{//...}
class GroundUnit extends Unit{//...}

class Knight extends GroundUnit implements Healable{}
class Giant extends GroundUnit implements Healable{}
class HogRider extends GroundUnit implements HogRidable, Healable{}
class BattleHealer extends GroundUnit implements Jumpable, Healable{}
```

- If all ground units are healable, it can be more simple.
```java
class GroundUnit extends Unit implements Healable{} //is this possible?
```

Below is a visualized version of class relations.
[![img]({{ "/assets/img/oop3-3.png"|absolute_url}})]({{ "/assets/img/oop3-3.png"|absolute_url}})

<br>
The point is, the healing feature is so smoothly added, without much altering to the existing classes!
This is the power of abstraction and interface-driven design. If I were to do in otherwise way, 

```java
class Knight extends GroundUnit{
    private Boolean canBeHealed;    //perhaps I would need something like this...
}
```

And what if I want the amount of healing to be different for the sizes of units? Then I would need to define heal points differently for ALL UNITS.
Below is a rough description of the method heal().

```java
void heal(Healable h){
    if(h instanceof GroundUnit){
        GroundUnit gu = (GroundUnit)h; //typecasting is necessary to use unique variables and methods of 'GroundUnit'
        while(gu.getCurrentHitPoint() <= gu.MaxHitPoint){ //current and max hitpoint concept are suddenly added for explanation
            if(gu.getSize().equals(SizeConstants.SMALL))
                gu.setHitPoint(gu.getCurrentHitPoint()+1);
            else if(...){
                //...
            }
        }
    }
}
```

Such is the power of interface! :D