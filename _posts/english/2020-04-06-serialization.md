---
layout: post
title:  "Serialization"
subtitle: 
description:
date:   2020-04-06 00:00:00 +0900
comments: true
image: /assets/img/dome.jpeg
optimized_image: /assets/img/dome.jpeg
category: java
tags: Java
author: tlonist
---

### Serialization

Can an object be stored inside of computer? Of course. **Serialization** helps this feature. 
In this posting, I will talk about 

- What serialization is.
- How it is done (with code examples)


### What is a serialization?

- Serialization is a process of writing an object into a data stream. To be more specific, it is a process of transforming the data stored in an object into serial data to be stored in a stream. The reverse process, writing serial data from stream into an object, is called **deserialization**.

[![img]({{ "/assets/img/serialization1.png"|absolute_url}})]({{ "/assets/img/serialization1.png"|absolute_url}})
- Note that methods and class variables(static) are NOT part of an object.

### How is serialization done?

#### 1. ObjectInputStream, ObjectOutputStream

- Here are two examples of writing object into serializable data, and the reverse of it.

```java
ObjectInputStream(InputStream in)
ObjectOutputStream(OutputStream out)

FileOutputStream fos = new FileOutputStream("objectfile.ser");
ObjectOutputStream out = new ObjectOutputStream(fos);

out.writeObject(new UserInfo());
```

```java
FileInputStream fis = new FileInputStream("objectfile.ser");
ObjectInputStream in = new ObjectInputStream(fis);

UserInfo info = (UserInfo)in.readObject();
```

The process of serialization/deserialization can usually take lots of time, since it has to convert all instance variables an object references. It is preferred to override 
```java
private void writeObject(ObjectOutputStream out)
private void readObject(ObjectInputStream in)
```
these methods for better performance.

#### 2. Making serializable classes

- Making a class serializable can be done by implemening **serializable** interface to the original class. 

```java
/*
 * Serializability of a class is enabled by the class implementing the
 * java.io.Serializable interface. Classes that do not implement this
 * interface will not have any of their state serialized or
 * deserialized.  All subtypes of a serializable class are themselves
 * serializable.  The serialization interface has no methods or fields
 * and serves only to identify the semantics of being serializable. <p>
 *
 * To allow subtypes of non-serializable classes to be serialized, the
 * subtype may assume responsibility for saving and restoring the
 * state of the supertype's public, protected, and (if accessible)
 * package fields.  The subtype may assume this responsibility only if
 * the class it extends has an accessible no-arg constructor to
 * initialize the class's state.  It is an error to declare a class
 * Serializable if this is not the case.  The error will be detected at
 * runtime. <p>
*/
public class UserInfo implements Serializable {
}
```
- How can implementing an empty class **serializable** make a class to be serializable? In short, it does not MAKE a class to be serializable, but instead, it MARKS the class is meant for serialization. For example, 

```java
    public static void main(String[] args) {
        UserInfo userInfo = new UserInfo();
        if(userInfo instanceof Serializable){
            System.out.println("It is serializable.");
        }else{
            System.out.println("It is not serializable.");
        }
    }

```

- If a class inherited a class which implemented Serializable, it doesn't need to implement Serializable again. However, 

```java
    public class SuperUserInfo {
        String name;
        String password;
    }

    public class UserInfo extends SuperUserInfo implements Serializable {
        int age;
        Object object; // WRONG!
        transient String gender;
    }

   public static void main(String[] args) {
        UserInfo userInfo = new UserInfo();
        try(FileOutputStream fileOutputStream = new FileOutputStream("serial.ser");){
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream);
            objectOutputStream.writeObject(userInfo);
        }catch (IOException i){
            System.out.println(i);
        }
    }
```
note that in case of above, only **age** is serializable, since the parent class SuperUserInfo is not serializable.
- The main method above throws exception, since **Object** is not serializable. However, changing the object to any serializable object will work. Such as **Object object = new String()**.
- **transient** renders the specific variable to be ommited from serialization/deserialization. 

#### 3. Let's do it

- Below is the actual implementation.

```java
    public static void main(String[] args) {
        UserInfo userInfo1 = new UserInfo("harry", "harry@tlonist.com", 12);
        UserInfo userInfo2 = new UserInfo("admin", "admin@tlonist.com", 32);
        ArrayList<UserInfo> list = new ArrayList<>();

        list.add(userInfo1);
        list.add(userInfo2);

        try(FileOutputStream fileOutputStream = new FileOutputStream("/Users/tlonist/Documents/UserInfo.ser");){
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream);

            objectOutputStream.writeObject(userInfo1);
            objectOutputStream.writeObject(userInfo2);
            objectOutputStream.writeObject(list);

            System.out.println("Serialization finished!");
        }catch (IOException i){
            System.out.println(i);
        }
    }
```

- After serialization is done, opening the UserInfo.ser file using textEdit...
[![img]({{ "/assets/img/serialization2.png"|absolute_url}})]({{ "/assets/img/serialization2.png"|absolute_url}})

- Deserialization is just the opposite of the above process.

```java
    try(FileInputStream fileInputStream = new FileInputStream("/Users/tlonist/Documents/UserInfo.ser");){
            ObjectInputStream in = new ObjectInputStream(fileInputStream);
            UserInfo userInfo1 = (UserInfo)in.readObject();
            UserInfo userInfo2 = (UserInfo)in.readObject();
            ArrayList list = (ArrayList)in.readObject();

            System.out.println(userInfo1.toString());
            System.out.println(userInfo2.toString());
            System.out.println(list);


        }catch(IOException | ClassNotFoundException e){
            System.out.println(e);
    }
```

```console
UserInfo{name='harry', email='harry@tlonist.com', age=12}
UserInfo{name='admin', email='admin@tlonist.com', age=32}
[UserInfo{name='harry', email='harry@tlonist.com', age=12}, UserInfo{name='admin', email='admin@tlonist.com', age=32}]
```
- Though the variable names don't need to match when serializing/deserializing, the order of them should match. The above example was serialized in order of **object, object, list**. Hence the deserialization should also be done in the order.

#### 4. Class version control of serializable classes.

- When deserializing a serialized object, one should always use the **same class**. If the class information is changed, it will cause ***InvalidClassException***. 
- Bear in mind that class variables and methods are NOT part of an actual object. Hence modifying them won't cause the above error. 
- You can set the version of a class manually by using a static variable like below.

```java
public class UserInfo implements Serializable {
    String name;
    int passcode;
    int age;

    static final long serialVersionUID = 1234567890123456789L; //SerialID

    public UserInfo(String name, int passcode, int age) {
        this.name = name;
        this.passcode = passcode;
        this.age = age;
    }
}
```
- Without the serialVersionUID(The name is reserved), the **InvalidClassException** will continue to pop up.



