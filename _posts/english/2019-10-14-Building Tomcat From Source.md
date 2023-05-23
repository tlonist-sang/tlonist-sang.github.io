---
layout: post
title:  "Building Tomcat from Source"
subtitle: "part 1"
description:
date:   2019-10-14 00:45:22 +0900
comments: true
image: /assets/img/writing.jpg
optimized_image: /assets/img/writing.jpg
category: server
tags: Server
author: tlonist
---

### Building Tomcat From Its Source - 1 

#### Downloading Tomcat Source and Opening it from an IDE

So you want to build Tomcat from the downloaded source src.zip? Here we go.
**Link**: [Download Source](https://tomcat.apache.org/download-90.cgi)

1. Download from Source Code Distributions. (I chose 'zip')

2. You can see the source code from IDE. 
    <img src="/assets/img/buildtomcat1.png" width="1200px">

3. (IntelliJ only) IntelliJ won't recognize the opened source as its source, unless you tell it to.
    ![img]({{ "/assets/img/buildtomcat2.png"|absolute_url}})
    Select the java directory, and choose 'source' to make it recognize as a Java source file.
    ![img]({{ "/assets/img/buildtomcat3.png"|absolute_url}})

4. Right after 3, you will see lots of red linings. This is because the project does not know which JDK should be used for the source code. Set the project SDK setting
    ![img]({{ "/assets/img/buildtomcat4.png"|absolute_url}})

5. ![img]({{ "/assets/img/buildtomcat5.png"|absolute_url}})
Now you are ready to edit the source as you please, and use it for your own joy.

