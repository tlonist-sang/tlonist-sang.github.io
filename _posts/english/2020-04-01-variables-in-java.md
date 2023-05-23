---
date: 2020-04-01 09:00:00 +0900
layout: post
title:  "Variables and Arrays"
subtitle: Review
description: a short review
image: /assets/img/array.jpeg
optimized_image: /assets/img/array.jpeg
category: java
tags: Java
author: tlonist
---

### Variables in Java

#### 1. Types

Primitive Types : Saves data literal. (boolean, byte, short, int, long, float, double, char)
Reference Types : Saves address of the data. (String, etc...)

Ranges of various variable types

|       | byte         | short          | int        | long         |
|-------|--------------|----------------|------------|--------------|
| size  | 1 byte       | 2 bytes        | 4 bytes    | 8 bytes      |
| range | -2^7 ~ 2^7-1 | -2^15 ~ 2^15-1 | -2^31~2^31 | -2^63~2^63-1 |


|       | boolean    | char                    | float           | double            |
|-------|------------|-------------------------|-----------------|-------------------|
| size  | 1 byte     | 2 bytes                 | 4 bytes         | 8 bytes           |
| range | true/false | 0~2^16-1(\u0000~\uffff) | -1.4E-45~3.4E38 | -4.9E-324~1.8E308 |

- All java operation is done among 'primitive' types.
- float and double have a concept of 'accuracy' denoting upto what digit the expression is hold accurate. (float is 7, double is 15). Though the size of float(double) is same as that of int(long), float(double) can express broader range of numbers by sacrificing its accuracy.

#### 2. default values

|         | byte | short | int | long |
|---------|------|-------|-----|------|
| default | 0    | 0     | 0   | 0L   |

|         | boolean | char   | float | double |
|---------|---------|--------|-------|--------|
| default | false   | \u0000 | 0.0f  | 0.0d   |

### Arrays

#### 1. Initialization
```java
    int[] score = new int[5];
```
- int[] score : defines a reference variable for the array. Array not yet made.
- new int[5] : creates an array with five integers whose address is then assigned to 'score'

#### 2. Copying Arrays

- System.arraycopy(arr, index_of_arr_to_start, newArr, index_of_newArr_to_start, arr.length);
```java
public static void main(String[] args){
    char[] alphabet = {'A', 'B'};
    char[] number = {'1','2','3'};

    char[] sum = new char[alphabet.length + number.length];
    System.arraycopy(alphabet, 0, sum, 0, alphabet.length);
    System.arraycopy(number, 0, sum, alphabet.length, number.length);
}
```

#### 3. N dimensional Arrays

- String [4][1][5][9] arr : String array of 4 dimension.
- int [3][4] numArr : int array of 2 dimension.

```java
    String[][][][] arr = new String[][][][]; //Error! Array initializer expected
    String[][][][] arr = new String[4][][][]; //O.K

    int[][] numArr = new int[3][4];
    for(int[] i : numArr){
        for(int j : i){
            System.out.println("number=>"+j);
        }
    }
```
- Note that when looping a multi-dimensional array, the reference variable holds the address of the outermost array as a reference. Hence for(int[] i : numArr).

- Diagram explanation below.
[![img]({{ "/assets/img/multiarr.png"|absolute_url}})]({{ "/assets/img/multiarr.png"|absolute_url}})

Read more - [Statics Variables in Java - Scaler Topic](https://www.scaler.com/topics/static-variable-in-java/)

