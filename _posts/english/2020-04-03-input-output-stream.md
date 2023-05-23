---
layout: post
title:  "Input IO in Java"
subtitle: "streams"
description:
date:   2020-04-03 00:00:00 +0900
comments: true
image: /assets/img/nz-house.jpeg
optimized_image: /assets/img/nz-house.jpeg
category: java
tags: Java
author: tlonist
---

### Input and Output in Java

***Stream is a passway for data transfer***
- InputStream : reads data from a source
- OutputStream : writes data to a destination

### 1. ByteStream

- FileInputStream
- ByteArrayInputStream
- PipedInputStream
- AudioInputStream

- In case when the file is made of characters, 'FileReader/Writer' can be used. All bytestreams have 1 byte unit, meaning they read and write by one byte. In Java, one character takes 2 bytes, hence using bytestream in processing characters may cause some troubles. Below is the example of using FileReader/Writer


```java
public class IOStream {
    public static void main(String[] args) {
        try(FileReader in = new FileReader(new File("input.txt"));
            FileWriter out = new FileWriter("output.txt")){ //try with source
            int c;
            while ((c = in.read()) != -1){
                out.write(c);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

- Executing above code in intelliJ may cause 'FileNotFoundException'.
[![img]({{ "/assets/img/filenotfound.png"|absolute_url}})]({{ "/assets/img/filenotfound.png"|absolute_url}})

- This is because the input.txt file is not referenced correctly when executing the compiled Java file. To tackle this problem, 

> 1. Find where .class file is located. In my case, it was out/IOSTREAM_PACKAGE/IOStream.class.
> 2. place input.txt file in the upper directory of where IOStream.class is located.
> 3. Run java IOSTREAM_PACKAGE.IOStream
> 4. Observe output.txt is created.

### 2. FilterStream

#### 2-1. BufferedStream
- FilterInputStream/FilterOutputStream are inherited from InputStream/OutputStream. They cannot perform I/O by themselves, hence necessitates byteStreams for operation. Let us look at the BufferedStreams. 

- Why is BufferedStream better? (https://medium.com/@isaacjumba/why-use-bufferedreader-and-bufferedwriter-classses-in-java-39074ee1a966) It is good because it can process data in bulk, instead of in one byte. Accessing data in disc, copying the byte, and converting it back to character per byte takes a lot of resources; instead if a bulk of data is copied at once, the unnecessary repetition of accessing the disc, converting them to characters etc is no more. Here is a code example.


```java
public static void main(String[] args) {
        try(FileInputStream in = new FileInputStream(new File("input.txt"));
            BufferedInputStream bufferedInputStream = new BufferedInputStream(in, 5);
            FileOutputStream out = new FileOutputStream("output.txt");
            BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(out, 5)){ //try with source
            int c;
            while ((c = bufferedInputStream.read()) != -1){
                bufferedOutputStream.write(c);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

#### 2-2 SequenceInputStream

- SequenceInputStream enables multiple inputstreams to be combined to one stream. Besides its constructor, other functionalities are identical. Here goes a code example.

```java
public static void main(String[] args) throws IOException {
        byte[] arr1 = {0,1,2};
        byte[] arr2 = {3,4,5};
        byte[] arr3 = {6,7,8};
        byte[] outSrc = null;

        Vector v = new Vector();
        v.add(new ByteArrayInputStream(arr1));
        v.add(new ByteArrayInputStream(arr2));
        v.add(new ByteArrayInputStream(arr3));

        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();

        int data = 0;

        try(SequenceInputStream sequenceInputStream = new SequenceInputStream(v.elements())){
            while((data = sequenceInputStream.read()) != -1){
                byteArrayOutputStream.write(data);
            }
        }

        outSrc = byteArrayOutputStream.toByteArray();
        System.out.println(Arrays.toString(outSrc));
    }
```

```console
    [0, 1, 2, 3, 4, 5, 6, 7, 8]
```

#### 2-3 PipedStream

- PipedReader/Writer is used when transferring data between two threads. Unlike other streams, pipedstreams connect input and output into one stream when exchanging the data.

- One thread calls 'connect()' function to connect with the other thread. After I/O is done, closing one stream automatically closes the other.

```java
public class PipedStream {
    public static void main(String[] args) {
        InputThread inputThread = new InputThread("in");
        OutputThread outputThread = new OutputThread("out");

        inputThread.connect(outputThread.getOutput());
        
        inputThread.start();
        outputThread.start();
    }
}
```

```java
public class InputThread extends Thread{
    PipedReader pipedReader = new PipedReader();
    StringWriter stringWriter = new StringWriter();

    InputThread(String name){
        super(name);
    }

    public void run(){
        try{
            int data = 0;
            while((data = pipedReader.read())!=-1){
                stringWriter.write(data);
            }
            System.out.println(getName() + " received : " + stringWriter.toString());
    }catch (IOException e) {
            e.printStackTrace();
        }
    }

    public PipedReader getInput(){
        return pipedReader;
    }

    public void connect(PipedWriter pipedWriter){
        try{
            pipedReader.connect(pipedWriter);
        }catch(IOException e){

        }
    }
}
```

```java
public class OutputThread extends Thread{

    PipedWriter pipedWriter = new PipedWriter();
    OutputThread(String name) {
        super(name);
    }

    public void run(){
        try{
            String message = "hello! sending..";
            System.out.println(getName() + " set : " + message);
            pipedWriter.write(message);
            pipedWriter.close();
        }catch(IOException e){

        }
    }

    public PipedWriter getOutput(){
        return pipedWriter;
    }

    public void connect(PipedReader pipedReader){
        try{
            pipedWriter.connect(pipedReader);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```console
out set : hello! sending..
in received : hello! sending..
```