---
layout: post
title:  "Factory pattern (1/2)"
subtitle: "with Java"
description:
date:   2020-02-12 00:45:22 +0900
comments: true
image: /assets/img/cow.jpeg
optimized_image: /assets/img/cow.jpeg
category: java
tags: Design Pattern
author: tlonist
---

## What is a factory pattern, and when can it be used?
There are plenty of explanations on what it is, and when it can be used; but I'm not so convinced as to why it solves the problem it claims to solve. This post will focus more on this part, while mentioning briefly about what the factory pattern is.

### - What is it?
What does a factory do? Simply put, it produces products in mass quantity. A client requests an order to a factory, and in return the factory produces products. The point is, the client **never makes the product by himself.** Instead, he delegates the creation of the product to the factory.

Exactly the same thing applies to the world of design pattern. Here goes example codes.

So instead of doing this,
```java
public class test {
    public static void main(String[] args) {
        Americano americano = new Americano();
        System.out.println(americano.getName());

        CaffeLatte caffeLatte = new CaffeLatte();
        System.out.println(caffeLatte.getName());
    }
```

Using **factory method pattern** will be something like this below.
```java
public class test {
    public static void main(String[] args) {
        CoffeeFactory coffeeFactory = new BasicCoffeeFactory();
        Coffee americano = coffeeFactory.getCoffee("Americano");
        Coffee caffeLatte = coffeeFactory.getCoffee("CaffeLatte");

        System.out.println(americano.getName());
        System.out.println(caffeLatte.getName());
    }
}
```


**Abstract factory pattern** can be applied when a higher level of abstraction is required
```java
public class BasicCoffeeFactory extends CoffeeFactory {
    AbstractCoffeeFactory abstractCoffeeFactory;
    public BasicCoffeeFactory(AbstractCoffeeFactory abstractCoffeeFactory){
        this.abstractCoffeeFactory = abstractCoffeeFactory;
    }

    @Override
    Coffee getCoffee(String type) {
        Coffee coffee = null;
        switch(type){
            case "Americano":
                coffee =  abstractCoffeeFactory.getCoffee("Americano");
                break;
            case "CaffeLatte":
                coffee =  abstractCoffeeFactory.getCoffee("CaffeLatte");
                break;
            default:
                coffee = new DripCoffee();
                break;
        }
        return coffee;
    }
}
```

```java
public class ItalianCoffeeFactory extends AbstractCoffeeFactory {
    @Override
    Coffee getCoffee(String coffeeType) {
        switch(coffeeType){
            case "Americano":
                return new ItalianCoffee();
            case "CaffeLatte":
                return new ItalianLatte();
            default:
                return new DripCoffee();
        }
    }
}
```

```java
public class KoreanCoffeeFactory extends AbstractCoffeeFactory {
    @Override
    Coffee getCoffee(String coffeeType) {
        switch(coffeeType){
            case "Americano":
                return new KoreanAmericano();
            case "CaffeLatte":
                return new KoreanLatte();
            default:
                return new DripCoffee();
        }
    }
}
```

```java
public class test {
    public static void main(String[] args) {
        BasicCoffeeFactory koreanCoffeeFactory = new BasicCoffeeFactory(new KoreanCoffeeFactory());
        BasicCoffeeFactory italianCoffeeFactory = new BasicCoffeeFactory(new ItalianCoffeeFactory());

        Coffee coffee1 = koreanCoffeeFactory.getCoffee("Americano");
        Coffee coffee2 = italianCoffeeFactory.getCoffee("Americano");

        System.out.println(coffee1.getName());
        System.out.println(coffee2.getName());
    }
}
```

```console
KoreanAmericano
ItalianCoffee
```

### - Why is it so good? When does it prove useful?
Okay, there is nothing so 'speicial' about factory pattern; there is no magic. It makes use of the good old interfaces, abstract methods and their implementations. I do not need to **new** it to create objects, but instead I had to write quite a lengthy lines for making the factories. 

So the question boils down to, is it worth it? From the simple example above, (and numerous other blog posts whose examples are all very simple as well) I cannot see much benefit of it. Perhaps 
1) The code has gotten more systematic. (You expect KoreanCoffeeFactory to produce all Koreanized-version of coffees)
2) and...

I think I've seen lots of these factories in **Spring source code**, so let me find evidences for its usefulness.. in the next posting!

