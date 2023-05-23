---
layout: post
title:  "React.js - simple app practice (1/4)"
subtitle: "Image fetching app, part1 - structure"
description:
date:   2020-01-08 00:45:22 +0900
comments: true
image: /assets/img/sweden.jpg
optimized_image: /assets/img/sweden.jpg
category: javascript
tags: JavaScript, React.js
author: tlonist
---

Thank God!!! my team has finally decided to overthrow our jQuery + pure html driven frontend HELL (lots and lots of files, each of which runs easily beyond 3000+ lines), and adopt React.js for refactoring. It has been like a year since I firstly got in touch with the library, and I'd like to re-do some of the practices I did while taking a udemy course on React.js. Here it goes.

The goal is rather simple. 
[![img]({{ "/assets/img/react-apple.png"|absolute_url}})]({{ "/assets/img/react-apple.png"|absolute_url}})

As can be seen from the image above, 
- User types a word in the search box. (presses enter)
- The word gets sent to a website for fetching images.
- Images pertaining to the word gets displayed.

Simple as it may sound, this exercise covers following important topics.
- **using props**
- **making api calls**
This is a very elementary exercise, and I will cover some other advanced topics (hopefully) as I get better at this.
***Entire code can be found from the source [Stephen Grider's React Course - pics](https://github.com/StephenGrider/redux-code/tree/master/pics)***


Creating the scaffold of a react application is easy. 
```javascript
create-react-app YOUR_REACT_APP_NAME //will just do!
```

Below is the structure of components that will consist the practice react app.
[![img]({{ "/assets/img/react-structure.png"|absolute_url}})]({{ "/assets/img/react-structure.png"|absolute_url}})

- APP => The entirity of simple app 
- SearchBar => input box that takes user input
- ImageList => div section that encompasses many images 
- ImageCard => individual images searched with user's input

What is different about using React.js from using jQuery + html is that it basically uses only **one** page to build the entire frontend. All I basically need is **index.js** into which I put all of my components in and out. If you look into index.js, 

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';

ReactDOM.render(
    <App />,
    document.querySelector("#root")
);
```
ReactDOM renders React codes into DOM elements so that they can be viewed with the browser. As can be noted, I have only one component, **App**, which contains all the components needed by the app inside. Let's look at the directory structure of the app. 

[![img]({{ "/assets/img/react-directories.png"|absolute_url}})]({{ "/assets/img/react-directories.png"|absolute_url}})

- Components will be rendered inside of index.js
- Api will be used somewhere inside of components to execute some actions.

With these in mind, let's go over the codes.


