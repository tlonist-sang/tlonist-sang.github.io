---
layout: post
title:  "JavaScript simple Array and Object manipulations"
subtitle: "Let's get it"
description:
date:   2020-03-05 00:45:22 +0900
comments: true
image: /assets/img/nz.jpeg
optimized_image: /assets/img/nz.jpeg
category: javascript
tags: javascript
author: tlonist
---

JavaScript is Very very fun actually! Look below :D

[![img]({{ "/assets/img/thx_js.jpeg"|absolute_url}})]({{ "/assets/img/thx_js.jpeg"|absolute_url}})


(1) Removing an element from an array <br>
(2) Adding an element to an array <br>
(3) Replacing an element in an array <br>

```javascript
    let arr = ['java', 'python', 'ruby'];
    let removing_element = arr.filter(element => element !== 'java')
    console.log(removing_element);

    let adding_element = [...arr, 'javascript']
    console.log(adding_element);

    let replacing_element = arr.map(element=>element === 'java'?'javascript':element)
    console.log(replacing_element);
```
```console
["python", "ruby"]
["java", "python", "ruby", "javascript"]
["javascript", "python", "ruby"]
```

(4) Removing an entry in an object <br>
(5) Adding an entry to an object <br>
(6) Updating an entry in an object <br>
```javascript
let state = {name:'harry', max_pullups:'21'}
let removing_value = {...state, name: undefined} 
console.log(removing_value);

let removing_value = _.omit(state, 'name') //this can be done with lodash!

let adding_new_entry = {...state, age:'99'}
console.log(adding_new_entry);

let updating_entry = {...state, name:'tlonist'}
console.log(updating_entry);
```
```console
{name: undefined, max_pullups: "21"}
{name: "tlonist", max_pullups: "21"}
{name: "harry", max_pullups: "21", age: "99"}
```

(7) Iterating keys in object <br>
(8) Iterating values in object <br>
(8) Iterating entries in object <br>
```javascript
let state = {'name':'alex Delarge', age:22, createdby:'anthony burgess'}

let arr_keys = Object.keys(state)
console.log(arr_keys);

let arr_values = Object.values(state)
console.log(arr_values);

let arr_entry = Object.entries(state)
console.log(arr_entry);
```
```console
["name", "age", "createdby"]
["alex Delarge", 22, "anthony burgess"]
[Array(2), Array(2), Array(2)]
```


[![img]({{ "/assets/img/thx_js1.png"|absolute_url}})]({{ "/assets/img/thx_js1.png"|absolute_url}})

https://medium.com/@darrion/ways-to-loop-through-an-object-in-javascript-622353049c7f