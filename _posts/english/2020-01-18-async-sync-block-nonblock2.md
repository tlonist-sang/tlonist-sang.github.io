---
layout: post
title:  "Blocking VS Non-blocking (2/2)"
subtitle: "Is sync == blocking and async == non-blocking ? (2/2)"
description:
date:   2020-01-21 00:45:22 +0900
comments: true
image: /assets/img/universe.jpeg
optimized_image: /assets/img/universe.jpeg
category: java
tags: Java, JavaScript
author: tlonist
---

In the previous post, I explored concepts and examples of synchronous and asynchronous models in Java and JavaScript. 
Here, I will write about blocking and non-blocking models. On surface, the equation sync : blocking = async : nonblocking seems to make sense, but they are NOT. Here goes the example.

- Blocking:
-----------

When I was little, there was a place called Jungle Gym; it was a literal paradise for children, teeming with fun rides and video games to play (it must have been a paradise for parents as well). Anyways, of many video games, the most popular one at that time was  the Lion King. But what was nasty about that game was that because it was so popular, I never got to play it. Whomever set on the small chair in front of that game was **blocking** other children from playing it; they had to wait until one lucky guy was fed up with playing the game.

- Non-blocking:
---------------

While the Lion King was a nasty, one-person overruling hopeless game for many, Sonic was different. What was special about this game was that more than one person could participate in **the middle**. If I'm playing Sonic (the character), other guy could join in by playing Tails. The process was **non-blocking** in a way, meaning I could keep on playing the game when another player joins in the middle.


> The gist is, whether a process is blocking or non-blocking is determined by the timing of the called function's return.
In a blocking process, the function called does NOT hand over the control to the calling fucntion until it finishes; in a non-blocking process, the called function hands over the control **right after it is called** to the calling function, so that the calling fucntion can do other works.

Blocking and non-blocking models are often described with I/O process. Let's look at code examples in the light of file I/O processes of Java and JavaScript.


### Java

[code reference](https://medium.com/coderscorner/tale-of-client-server-and-socket-a6ef54a74763)

What does it essentially mean by 'blocking' then? Considering that every single line of codes is run within a thread, **blocking** would mean **not letting a thread do other work** while the blocking job is being done. 

[![img]({{ "/assets/img/block2.png"|absolute_url}})]({{ "/assets/img/block2.png"|absolute_url}})

- The diagram above shows the process of blocking I/O between a client and a server. When a client-server is communicating with blocking I/O (model? protocol?), the thread allocated for the communication is totally blocked, meaning it cannot do other works other than that. Let's look into a code example. It is sudo-codey, not a literal code that can be executed with results.

```java
ServerSocket serverSocket = new ServerSocket(port_number); // -- (1)
Socket clientSocket = serverSocket.accept(); // -- (2)

/*
 (1) : Creates a socket at server-side for communication, allocating a specific port 
 (2) : ServerSocket accetps, meaning it will wait for client requests incoming through port_number.
*/

BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream())); // -- (3)
PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true); // -- (4)

/*
 (3) : InputStreamReader is wrapped around clientSocket.getInputStream, reading what client has sent.
 (4) : OutputStream is allocated for clientSocket.getOutputStream, to wrtie out reponse from the server to client.
*/

String request, response;
while ((request = in.readLine()) != null) { // -- (5)
  response = processRequest(request);
  out.println(response);
  if ("Done".equals(request)) { 
    break;
  }
}
/*
 (5) : Notice that the while loop doesn't break until it hits IF clause below. Making the process a 'blocking' thing.
*/
```

So when a server has to deal with lots of clients in blocking I/O way, it will look something like below.
[![img]({{ "/assets/img/block1.png"|absolute_url}})]({{ "/assets/img/block1.png"|absolute_url}})

While it is clearly an easy way to program, there are some noticeable disadvantages.

- Server has to create threads as many as the number of clients trying to communicate. This lowers the performance of the server as threads will eat up huge amount of stack memory, not to mention that resources will be used in switching contexts among them.
- Server resource will be wasted if non-responding clients do not end the communication by "Done".


So we have an async model this time. It is a little more complicated implementation. Let us bear in mind that the non-blocking means **the called function returning right away so that the calling function can do other work**.

[![img]({{ "/assets/img/block4.png"|absolute_url}})]({{ "/assets/img/block4.png"|absolute_url}})

In non-blocking I/O model of Java, there is a special thread called **selector**, that governs all of the incoming and outgoing communications of multiple clients. How is this possible?

```java
Selector selector = Selector.open();

ServerSocketChannel serverChannel = ServerSocketChannel.open();
serverChannel.configureBlocking(false); // -- (1) 

// (1) : Open a selector and a channel for communication. (It is not a socket anymore!)

InetSocketAddress hostAddress = new InetSocketAddress(hostname, portNumber);
serverChannel.bind(hostAddress);
serverChannel.register(selector, SelectionKey.OP_ACCEPT); // -- (2)

// (2) : Bind address, port, and a selector to the channel.

while (true) {
   int readyCount = selector.select(); // -- (3)
   if (readyCount == 0) { // -- (3)
      continue;
   }
    Set<SelectionKey> readyKeys = selector.selectedKeys();
    Iterator iterator = readyKeys.iterator();
    while (iterator.hasNext()) { // -- (4)
        SelectionKey key = iterator.next(); // -- (4)
        iterator.remove();
    
/*
 (3) : Selector searches out for the ready channel, and select them. If none is ready, just keep on doing it. 
       This somewhat resembles the eventListener, and is the KEY for the concept of non-blocking IO.

 (4) : Once channels are ready, iterate over them and remove the keys
*/
        if (key.isAcceptable()) {
            ServerSocketChannel server = (ServerSocketChannel)  key.channel();    
            SocketChannel client = server.accept();
            client.configureBlocking(false);
            client.register(selector, SelectionKey.OP_READ); // -- (5)
            continue;
        }

// (5) : If the key is acceptable, accept the channel for communication and configure it 'non-blocking' (hmm.. blocking is also possible). Then register it for 'READ' or 'WRITE'
    }
}
```
Okay, that was rather long. **BREAK TIME**

```java

    if (key.isReadable()) {
      SocketChannel client = (SocketChannel) key.channel();
      int BUFFER_SIZE = 1024;
      ByteBuffer buffer = ByteBuffer.allocate(BUFFER_SIZE);
      try {
        client.read(buffer);
      }
      catch (Exception e) {
        e.printStackTrace();
        continue;
      } // -- (6)

      if (key.isWritable()) {
        SocketChannel client = (SocketChannel) key.channel();
        SocketAddress address = new InetSocketAddress(hostname, portnumber);
        SocketChannel client = SocketChannel.open(address);

        ByteBuffer buffer = ByteBuffer.allocate(74);

        buffer.put(msg.getBytes());
        buffer.flip();
        client.write(buffer);
      } // -- (7)

/*
 (6) : If the key is for reading, set up the socket channel for reading and read it.
 (7) : If it is for writing, write to the client using buffer!
*/
```

- The key concept above is that the **selector** can select a set of **ready channels** for listening; and once it is done reading or writing, it **removes the key** from key list to accept other keys in the future. I haven't implemented a non-blocking I/O myself, but the concept is so. 

- It is worth questioning, how is this non-blocking? Well, comparing this with the former blocking I/O, in non-blocking I/O, multiple read/write actions from multiple clients can be performed in a **single thread**. Of course, the while when the actual reading and writing is being performed the function cannot return right away, but not one server-client communication is taking over the entire thread, blocking other server-client communications from operating. 


### JavaScript

- JavaScript example is rather easy to understand. 

```javascript
    const fs = require('fs');
    const data = fs.readFileSync('./example.txt'); // blocks here until file is read
    console.log(data.toString());
    doSomething();

    function doSomething(){
    console.log("hello world!");
    }
```

```console
power overwhelming
show me the money
operation cwal
black sheep wall
something for nothing
the gathering

###hello world!###
```

```javascript
    const fs = require('fs');
    fs.readFile('./example.txt', (err, data) => {
    if (err) throw err;
    console.log(data.toString());
    });

    doSomething();
    function doSomething(){
    console.log("###hello world!###");
    }
```
```console
    ###hello world!###

    power overwhelming
    show me the money
    operation cwal
    black sheep wall
    something for nothing
    the gathering
```

- By using the callback pattern, the non-blocking I/O can be easily demonstrated.

In this posting, I covered...
- What is blocking and non-blocking models in programming
- Examples from Java side
- Examples from JavaScript side.

Questions remain, *** Can it be Synchronous AND Non-blocking, and Asynchronous AND Blocking?
- Why not? Let's think about some concrete examples.. in the next post.

references
- [https://medium.com/coderscorner/tale-of-client-server-and-socket-a6ef54a74763](https://medium.com/coderscorner/tale-of-client-server-and-socket-a6ef54a74763)

- [https://nodejs.org/de/docs/guides/blocking-vs-non-blocking/](https://nodejs.org/de/docs/guides/blocking-vs-non-blocking/)