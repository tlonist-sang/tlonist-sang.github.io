---
layout: post
title:  "Annotation in Java"
subtitle: "custom annotation"
description:
date:   2020-04-07 00:00:00 +0900
comments: true
image: /assets/img/coffee1.jpeg
optimized_image: /assets/img/coffee1.jpeg
category: java
tags: Java
author: tlonist
---


### Prelude
- When using Spring framework or other Java-based libraries, one encounters a lot of annotations. Annotations alone cannot do anything for the operation of code, but annotated data can be used using Java Reflections API at runtime or using Annotation Processors at compile time. 
[https://medium.com/@nadundesilva/java-custom-annotation-99eaf3b6cd7e]
[https://www.baeldung.com/java-custom-annotation]

### Annotations
#### 1. Standard annotations

- The table below shows the list of standard annotations provided by Java. The ones with asterisk(*) denote meta-annotations used for creating annotations. Details will be studied soon when making my own custom annotation.

| Annotations          | Explanation                                               |
|----------------------|-----------------------------------------------------------|
| @Override            | Tell compiler it is overriding method                     |
| @Deprecated          | Recommend not to use                                      |
| @SuppressWarning     | Ignore certain warning messages                           |
| @SafeVarargs         | Used for Generics type variable                           |
| @FunctionalInterface | Signify functional interface                              |
| @Native              | Used for constants referenced by native method            |
| @Target*             | Set applicable targets where annotation should be applied |
| @Documented*         | Include annotation data in document written in javadoc    |
| @Inherited*          | Make annotation to be inherited to child class            |
| @Retention*          | Set the range of influence of annotation                  |
| @Repeatable*         | Enable repeated application of annotation                 |


#### 2. Making Custom annotations - beginning

- Creating a custom annotation is almost identical to creating an interface. Use @ in front of the name of annotation, and declare methods with a specfic return type like below. Available members of annotation are primitive, String, an Enum, another Annotation, Class, and an array of any of the above.[https://docs.oracle.com/javase/specs/jls/se8/html/jls-9.html#jls-9.6.1]

```java
public @interface UserInfo {
    String writtenBy();
    String[] testTools();
    WrittenDate writtenDate(); //another custom annotation
    CareerLevel careerLevel(); //enum value
}

public @interface WrittenDate {
}

public enum CareerLevel {
    Junior,
    Senior,
    Expert,
    Leader
}
```

- I can set default values using **default** keyword.
```java
    public @interface UserInfo {
        String writtenBy() default "admin";
        String[] testTools() default "intelliJ";
        WrittenDate writtenDate();
        CareerLevel careerLevel() default CareerLevel.Junior;
    }

    @UserInfo(writtenDate = @WrittenDate(date = "12:00:00"))
    public void testMethod1(){
    }
```
- The **testMethod1** above has default annotation elements 'admin', 'intellij' and 'CareerLevel.Junior'.
- Some annotations are called **Marker Annotation**. They do not have any elements. Just like **Serializable** and **Cloneable** interfaces, they are used to denote certain marks on a class. One very representative such annotation is @Override

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

#### 3. Making Custom Annotations - JsonSerializer

- This example is an exact replica of Baeldung post [https://www.baeldung.com/java-custom-annotation]. I added some comments for my own understanding.

- The goal of this custom annotation is to serialize an object into a JSON string. In doing so, three customized annotations will be used.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface JsonSerializable {
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface JsonElement {
    String key() default "";
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Init {
}
```

- Define a new class **Person** that makes use of the declared annotations. Make sure to put annotations according to their designated targets.

```java
@JsonSerializable
public class Person {

    @JsonElement
    private String firstName;

    @JsonElement
    private String lastName;

    @JsonElement(key = "personAge")
    private String age;

    private String address;

    public Person(String firstName, String lastName, String age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }

    @Init
    private void initNames() {
        this.firstName = this.firstName.substring(0, 1).toUpperCase()
                + this.firstName.substring(1);
        this.lastName = this.lastName.substring(0, 1).toUpperCase()
                + this.lastName.substring(1);
    }
}
```

- Annotations are nothing but markers if they are not interpreted to carry out certain operations. Here **ObjectToJsonConverter** does such work by using Java Reflections. 

```java

public class ObjectToJsonConverter {

    public String convertToJson(Object object) throws JsonSerializationException {
        try{
            checkIfSerializable(object);
            initializeObject(object);
            return getJsonString(object);

        }catch(Exception e){
            throw new JsonSerializationException(e.getMessage());
        }
    }

    private void checkIfSerializable(Object object) throws JsonSerializationException{
        if(Objects.isNull(object)){
            throw new JsonSerializationException("Can't serialize a null object");
        }
        Class<?> clazz = object.getClass();
        if(!clazz.isAnnotationPresent(JsonSerializable.class)){
            throw new JsonSerializationException("The class " + clazz.getSimpleName() + " is not annotated with JsonSerializable");
        }
    }

    private void initializeObject(Object object) throws IllegalAccessException, IllegalArgumentException, InvocationTargetException {
        Class<?> clazz = object.getClass();
        for(Method method : clazz.getDeclaredMethods()){
            if(method.isAnnotationPresent(Init.class)){
                method.setAccessible(true);
                method.invoke(object);
            }
        }
    }

    private String getJsonString(Object object) throws IllegalAccessException {
        Class<?> clazz = object.getClass();
        Map<String, String> jsonElementMap = new HashMap<>();
        for(Field field : clazz.getDeclaredFields()){
            field.setAccessible(true);
            if (field.isAnnotationPresent(JsonElement.class)) {
                jsonElementMap.put(getKey(field), (String) field.get(object));
            }
        }
        String jsonString = jsonElementMap.entrySet()
                .stream()
                .map(entry -> "\"" + entry.getKey() + "\":\"" + entry.getValue() + "\"")
                .collect(Collectors.joining(","));
        return "{" + jsonString + "}";
    }

    private String getKey(Field field) {
        String value = field.getAnnotation(JsonElement.class)
                .key();
        return value.isEmpty() ? field.getName() : value;
    }
}
```

- Finally, here goes the test.

```java
    @Test
    public void annotationTest() throws JsonSerializationException{
        Person person = new Person("Harry", "Kim", "99");
        ObjectToJsonConverter serializer = new ObjectToJsonConverter();
        String jsonString = serializer.convertToJson(person);
        System.out.println(jsonString);
    }
```

- and I get
```console
{"personAge":"99","firstName":"Harry","lastName":"Kim"}
```
, which is a desired output.
Next posting will be about Java Reflection APIs!!