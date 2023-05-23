---
layout: post
title:  "Building Tomcat from source"
subtitle: "part 2"
date:   2019-10-14 00:45:22 +0900
image: /assets/img/writing.jpg
optimized_image: /assets/img/writing.jpg
comments: true
category: server
tags: Server
author: tlonist
---

####Building Tomcat From Its Source - 2 
#####Downloading Tomcat Source and Opening it from an IDE

There can be various ways to build Tomcat, and I'll follow the most orthodox approach- from Apache Tomcat Manual. To build it, you need to have Ant. (link provided below)
**Link(Manual)**: https://tomcat.apache.org/tomcat-9.0-doc/building.html
**Link(Ant)**: https://ant.apache.org/bindownload.cgi


1. ![img]({{ "/assets/img/buildtomcat21.png"|absolute_url}})
As noted above, you need to set ant.home so that the ant command can be executed from the command window. Make sure to add them to the **environment variables.**. One to the $PATH, the other as ANT_HOME.

2. After setting them, you can type ant -version to see if its correctly applied.
![img]({{ "/assets/img/buildtomcat22.png"|absolute_url}})

3. Go to the downloaded Tomcat Source, enter one depth, find and rename **build.properteis.default** to **build.properties**

4. Set proxy setting if needed, else you can just leave it as it is.

5. Open command console, go to the directory where the **build.properties** file is located. Then, simply type ant.
![img]({{ "/assets/img/buildtomcat23.png"|absolute_url}})
You will see something going one like below.
![img]({{ "/assets/img/buildtomcat24.png"|absolute_url}})


When build is successful, Congratulations! you've just built a usable Tomcat from its source!
![img]({{ "/assets/img/buildtomcat25.png"|absolute_url}})