---
layout: post
title:  "require and import, what is the difference?"
subtitle: "brief mechanism underneath"
description:
date:   2020-07-30 00:00:00 +0900
tags: javascript
comments: True
image: /assets/img/babel.png
optimized_image: /assets/img/babel.jpeg
author: tlonist
category: JavaScript
---

### Prelude

I've been following up with a web project on Udemy by Stephen Grider, [node-with-react](https://github.com/tlonist-sang/nodereact). If there is an undeniable fact regarding JavaScript and the whole environment around it, it would be that the language is in its heydays. Anyways, having been more of a Java guy, there were lots of confusions as to what is the proper way to do a certain thing with JavaScript.

For instance, there were times when I used **import**, and there were times when I used **require**. Sometimes I used **module.exports**, and sometimes **export**. A [perfunctory search](https://stackoverflow.com/questions/46677752/the-difference-between-requirex-and-import-x/46677972) taught me that require/module.export is for commonJS, whereas import/export is for ES6. But what the heck, I've been using each from both quite interchangeably, and to my knowledge I've never explicitly declared any one of commonJS or ES6 when developing with node.js

This post is to make myself 100% confident on topics above, plus some micellaneous facts that I want to clarify when using node.js.

### 1. CommonJS and ES6
- [CommonJS](http://www.commonjs.org/) is a standard specification for JavaScript, especially catering for the need for modularization, which is a crucial concept for server development with JavaScript. Its original name was ServerJS, serving to its original purpose.
    - Facts (please criticize!)
        - **Node.js** is an implementation of **CommonJS**.
        - Modularization is comprised of the following three properties: scope, definition (using Module.exports), and usage (using require).
        - CommonJS acts on a premise that all of the modularized files are available in local disk, readily available whenever required. 
        - The above premise gave birth to AMD(Asynchronous Module definition), branched from CommonJS to focus on asynchronous modularization of JavaScript development.

- ES6, a.k.a [ECMAScript6](https://en.wikipedia.org/wiki/ECMAScript) (European Computer Manufacturers Association, what a name..!) is a script language. It is arguably a **subset** of JavaScript; JavaScript has its core as ECMAScript, and has more functions. There are many new features as introduced in [here](http://es6-features.org/#Constants)
    - Facts
        - ActionScript and JScript are also built upon ECMAScript; hence they are JavaScript's siblings.
        - As it is a standard, it updates and the latest as of now is ECMAScript 2020 (released in June). It keeps on adding new features.

- Having said this far, let's tackle the real question.    

### 2. How is [require] different from [import]?

> Summary : **require** is from CommonJS, **import** is from ECMAScript.

- Require : More details!
    - (Require)[https://nodejs.org/en/knowledge/getting-started/what-is-require/] : It is a function used to include modules from separate files. It reads a JavaScript file, executes it, and returns the export object. Here is a simple example.

    ```javascript
    //test.js
    exports.hello ='hello world';
    exports.say = function(){
        console.log('good morning');
    }
    function call(){
        console.log('Is this called?');
    }
    call();
    ```

    ```javascript
    //index.js
    const test = require('./test');
    console.log(test.hello);
    test.say();
    ```

    - The console result will be,
    ```console
    Is this called?
    hello world
    good morning
    ```

    - You can also use module.exports instead of exports. In this case, you can only export one functionality defined within module.exports.
    ```javascript
    //test2.js
    module.exports = function(){
    console.log('one module exports');
    ```

    ```javascript
    //index.js
    const test2 = require('./test2');
    test2();
    ```

    ```console
    one module exports
    ```
    [![img]({{ "/assets/img/require.png"|absolute_url}})]({{ "/assets/img/require.png"|absolute_url}})
    - As can be noted from the IDE itself(I'm using webstorm), the **module.exports** only has one component but none others, even when other functions or variables are defined within the test2.js file.

- Import:
    - Import is a ECMAScript6 syntax. Technically, import and require can both be used in one file. **Import** will be transformed into **require** by transpilers like babel. (explained here)[https://stackoverflow.com/questions/31354559/using-node-js-require-vs-es6-import-export]. Below is a code example.

    - One important thing to remember is that **require paris with exports (module.exports),** and **import with export, ES6**. Hence something like below doesn't work.
    ```javascript
      //test.js
    exports.hello ='hello world';
    exports.say = function(){
        console.log('good morning');
    }
    function call(){
        console.log('Is this called?');
    }
    call();
    ```

    ```javascript
    //index.js
    import test from './test';
    console.log(test.hello); // ERROR!
    test.say(); // ERROR!
    ```

    - You have to use **export** when using import.

    ```javascript
    //testImport.js
    export function sayHello(){
        console.log('Hi there!');
    }

    export function sayBye(){
        console.log('Bye!');
    }

    const sayHelloWorld = () => {
        console.log('Hello world!');
    }
    export {sayHelloWorld};
    ```

    ```javascript
    //index.js
    import {sayHello, sayBye} from './test'; //casual destructuring supported.
    sayHello(); //Hi there!
    sayBye(); //Bye!

    import * as greetings from './test';
    greetings.sayHelloWorld(); //Hello world!
    ```

    - Note that ECMAScript6 syntax is not yet officially supported by Node.js. Refer to (this link)[https://stackoverflow.com/questions/39436322/node-js-syntaxerror-unexpected-token-import] for enabling ES6 syntax within Node.js



### 3. Compare and Contranst [TO BE ADDED]

**Compare**
- Both are used to import modularized js files. 

**Contrast**
- 'require' loads all contents from the referenced js file; 'import' can selectively import.
- 'require' is synchronous; 'import' is 'asynchronous' (but if used in Node.js, import is translated into 'require', making it synchronous).

I will add how require and import works in more details.


### Wrapping up

- require and import stem from different roots of JavaScript standards.
- require - module.exports/exports, import - export
- require(CommonJS) is used in Node.js. Import can be used as well, but better have a consistency in a project.
- require and import are structurally different.
