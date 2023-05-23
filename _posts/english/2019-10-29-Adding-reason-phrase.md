---
layout: post
title:  "Tomcat 9 - Adding reason phrase 200 OK"
date:   2019-10-29 00:45:22 +0900
image: /assets/img/bear.jpg
optimized_image: /assets/img/bear.jpg
comments: true
category: server
tags: Server
author: tlonist
---

#### Abstract
From Tomcat 8.5 on, the **'reason phrase'**, which states some additional information to the incoming request according to the status code, is omitted. Not only did I feel frustrated for Apache developers who decided not to leave this even as an option, it was an acutal headache for me because the absence of the reason phrase was causing some serious issues for legacy projects. 

There were many people who were suffering from the same cause. Here are some examples.
    [https://stackoverflow.com/questions/40522145/javatomcat-9-status-code-200-but-response-message-null](https://stackoverflow.com/questions/40522145/javatomcat-9-status-code-200-but-response-message-null)<br/>
    [http://tomcat.10.x6.nabble.com/Bug-60362-New-Missing-reason-phrase-in-response-td5057191i40.html](http://tomcat.10.x6.nabble.com/Bug-60362-New-Missing-reason-phrase-in-response-td5057191i40.html)<br/>
    [ttps://superuser.com/questions/1201742/tomcat-reason-phrase](https://superuser.com/questions/1201742/tomcat-reason-phrase)<br/>
    ... etc


Here I'd like to share how to add the reason phrase manually, by manipulating Tomcat 9 source code itself. 
>The purpose of this article is to show that it is possible; and distribution of edited opensource project may not be permitted and is out of boundary of my knowledge. 

#### 1. Download Tomcat 9 Source code
- Download Tomcat 9 Source from apache Tomcat website (https://tomcat.apache.org/download-90.cgi)


#### 2. Download Tomcat 8(or 8.5) Source code 
- Download Tomcat 8 Source from apache Tomcat website (https://tomcat.apache.org/download-80.cgi)
#### 3. Observe the difference near 'reason phrase'

- Now it gets interesting. Using your IDE or whatever tool you have, try to observe the differences in code where 'reason phrase' appears.

###### Tomcat 8.5 
```java
public void sendStatus() {
        // Write protocol name
        write(Constants.HTTP_11_BYTES);
        headerBuffer.put(Constants.SP);

        // Write status code
        int status = response.getStatus();
        switch (status) {
        case 200:
            write(Constants._200_BYTES);
            break;
        case 400:
            write(Constants._400_BYTES);
            break;
        case 404:
            write(Constants._404_BYTES);
            break;
        default:
            write(status);
        }

        headerBuffer.put(Constants.SP);

        if (sendReasonPhrase) {
            // Write message
            String message = null;
            if (org.apache.coyote.Constants.USE_CUSTOM_STATUS_MSG_IN_HEADER &&
                    HttpMessages.isSafeInHttpHeader(response.getMessage())) {
                message = response.getMessage();
            }
            if (message == null) {
                write(HttpMessages.getInstance(
                        response.getLocale()).getMessage(status));
            } else {
                write(message);
            }
        } else {
            // The reason phrase is optional but the space before it is not. Skip
            // sending the reason phrase. Clients should ignore it (RFC 7230) and it
            // just wastes bytes.
        }

        headerBuffer.put(Constants.CR).put(Constants.LF);
    }
```
- Note the <mark>sendReasonPhrase</mark> part. For 8.5, sending reason phrase is optional and can be manipulated from <mark>server.xml</mark> file. (refer to [link](https://tomcat.apache.org/tomcat-8.5-doc/config/http.html)) But from Tomcat 9 on, it is no longer supported. 

###### Tomcat 9.0.26 
``` java
 public void sendStatus() {
        // Write protocol name
        write(Constants.HTTP_11_BYTES);
        headerBuffer.put(Constants.SP);

        // Write status code
        int status = response.getStatus();
        switch (status) {
        case 200:
            write(Constants._200_BYTES);
            break;
        case 400:
            write(Constants._400_BYTES);
            break;
        case 404:
            write(Constants._404_BYTES);
            break;
        default:
            write(status);
        }

        headerBuffer.put(Constants.SP);

        // The reason phrase is optional but the space before it is not. Skip
        // sending the reason phrase. Clients should ignore it (RFC 7230) and it
        // just wastes bytes.

        headerBuffer.put(Constants.CR).put(Constants.LF);
    }
```

- Contrasting the above two code snippets, you can notice that the entire part where it says <mark>sendReasonPhrase</mark> is missing in Tomcat 9. Your job is to restore this part, together with all the dependencies that are missing. 

[![img]({{ "/assets/img/tomcat9_red.png"|absolute_url}})]({{ "/assets/img/tomcat9_red.png"|absolute_url}})
- Now let's do something about'em.

#### 4. Add missing classes and variables

- HttpMessages (org.apache.tomcat.util.http)
[![img]({{ "/assets/img/tomcat9_httpmessages.png"|absolute_url}})]({{ "/assets/img/tomcat9_httpmessages.png"|absolute_url}})

- Constants (org.apache.coyote)
[![img]({{ "/assets/img/tomcat9_constants.png"|absolute_url}})]({{ "/assets/img/tomcat9_constants.png"|absolute_url}})

- write function (overloading)

- I've been trying this on my Macbook. Previously when I was testing this in my Windows desktop it was working smoothly, but somehow the Mac version is not. My IntelliJ is producing errors like below every time I try to launch my simple Spring MVC controller.
[![img]({{ "/assets/img/tomcat9_error.png"|absolute_url}})]({{ "/assets/img/tomcat9_error.png"|absolute_url}})

- I read from somewhere that the directory system of linux(and Mac) is different from that of Windows, hence causing this kind of error. In fact, when I copied the exact file to Windows and tested it, it was working like a charm.
- To tackle this problem anyhow (it is absurd that source-built tomcat file cannot be run in OS other than Windows), I reviewd the log (Help -> Show log in Finder) and found out that I was missing a logging library.
[![img]({{ "/assets/img/tomcat9_missing_library.png"|absolute_url}})]({{ "/assets/img/tomcat9_missing_library.png"|absolute_url}})

``` console
 <dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.8.0-beta2</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
```
[![img]({{ "/assets/img/tomcat9_add_library.png"|absolute_url}})]({{ "/assets/img/tomcat9_add_library.png"|absolute_url}})

- It is not working; and something is terribly wrong, and I decided firmly that I won't spend more than 30 minutes grappling with this bastard.
>The problem was from an awefully simple mistake.
>##### I downloaded src.zip, not src.tar.gz.
>There was a reason why Apache Tomcat was provided with two different versions after all. The tomcat directory structure must have been different, something that >cannot be easily manipulated by changing directory structures of visible files.

#### 5. Build modified Tomcat 9 and test if it's working

- Now, go to the Tomcat source directory, and rebuild the project using ant command. 

**Result**
[![img]({{ "/assets/img/tomcat9_200notworking.png"|absolute_url}})]({{ "/assets/img/tomcat9_200notworking.png"|absolute_url}})

- We definited expected something like this.
[![img]({{ "/assets/img/tomcat9_200working.png"|absolute_url}})]({{ "/assets/img/tomcat9_200working.png"|absolute_url}})

- Let's examine why. 
- If you look carefully into the class HttpMessages.java, you'll find that the actual OK code will be inserted with *getMessage* fucntion.

```java
public String getMessage(int status) {
        // method from Response.

        // Does HTTP requires/allow international messages or
        // are pre-defined? The user doesn't see them most of the time
        switch( status ) {
        case 200:
            if(st_200 == null ) {
                st_200 = sm.getString("sc.200");
            }
            return st_200;
```

- The string OK is picked up from a variable *sm*, (StringManager). You can simply search from **sc.200** from the whole package 8.5 or lower, to see where the key value is mapped into in the project. 
[![img]({{ "/assets/img/tomcat9_findsc200.png"|absolute_url}})]({{ "/assets/img/tomcat9_findsc200.png"|absolute_url}})


- Copy the entire package under the right position.
[![img]({{ "/assets/img/tomcat9_copypackage.png"|absolute_url}})]({{ "/assets/img/tomcat9_copypackage.png"|absolute_url}})

- Rebuild using ant, and enjoy the victory!
[![img]({{ "/assets/img/tomcat9_victory.png"|absolute_url}})]({{ "/assets/img/tomcat9_victory.png"|absolute_url}})


#### Concluding...

Though I was using Tomcat almost on daily basis with my company's project, I've never actually looked into its source code until this problem was nagging me. OpenSource projects, especially something as big and historical as Tomcat, has somehow been intimidating for me, not just because of its sheer lines of codes, but because its enigmatic(without even looking into them in details!) comments and interfaces. This is a small thing, really; and I might have done something wrong in the process. But it nontheless made me realize that I do not need to run away from intimidating looking codes anymore!