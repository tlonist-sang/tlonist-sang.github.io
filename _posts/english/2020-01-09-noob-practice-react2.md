---
layout: post
title:  "React.js - simple app practice (2/4)"
subtitle: "Image fetching app, part2 - implementation"
description:
date:   2020-01-09 00:45:22 +0900
comments: true
image: /assets/img/water.jpg
optimized_image: /assets/img/water.jpg
category: javascript
tags: JavaScript, React.js
author: tlonist
---

Ok, so let's begin with a simple search bar. 
Primarily, what a search bar has to do is the following.

- get user input.
- search for images according to the input.
- deliver the outcome of the search to other component for display.

Then let me define what functional modules need to be implemented to do so.

- get user input => **HTML input tag**
- search for images according to the input. => **submit a form request** (it doesn't have to be a form. it can be buttons, checkboxes or whatever.) 
- deliver the outcome of the search to other component for display. => **props**



There are always more than one way of writing a react code. Largely, I know that I can choose between the function based code and class based code. Let me choose the latter.

```javascript
import React from 'react';
class SearchBar extends React.component{
    render(){ //key method that 'renders' what the function itself returns
        return(
            <div>
                <form>
                    <input type="text" />
                </form>
            </div>
        );
    }
}
```

This is the most simple structure of the component; it return a div with form which has an input box inside.
Let's add a method that will be called when the form is finally submitted by my user.

```javascript
import React from 'react';
class SearchBar extends React.component{
    state = {userInput:""}
    
    onFormSubmit = (e)=>{
        e.preventDefault(); // -- (1)
        // API call -- (2)
    }
    
    render(){ 
        return( 
            <div>
                <form onSubmit={this.onFormSubmit}> 
                    <input type="text" />
                </form>
            </div>
        );
    }
}
```
- (1) When the user presses enter after inputing keywords, the function designated in 'onSubmit' is invoked.
      In this case, I did not specify any 'action' either - meaning I just want the form to execute 'onFormSubmit' function upon hitting enter. Not submitting any form to any where. Hence I used **e.preventDefault** to prevent what onSubmit event is designed to do!

- (2) What should be done next? of course calling the API with the keyword of user's choice. Wait.. but really?

Assume that the above process takes place like this.
[![img]({{ "/assets/img/react2-searchbar.png"|absolute_url}})]({{ "/assets/img/react2-searchbar.png"|absolute_url}})

The question is, How can I deliver the retrieved data to the sibling component. The diagram from previous post clearly shows that the SearchBar component and ImageList component are independent of each other. Hence they can't be communicated directly using props.

What can be done is that, upon calling the onSubmit event, it can call a method passed down from its parent component, App. How? by passing from App a method that will be called in SearchBar as a prop. All I have left is to make that passed on method (from App to SearchBar) to fire an API call to the API server.

[![img]({{ "/assets/img/react2-suggestion1.png"|absolute_url}})]({{ "/assets/img/react2-suggestion1.png"|absolute_url}})


```javascript
class App extends React.Component{
    onSearchSubmit = async (userInput) =>{ // -- (4)
        //API call
    }
    render(){
        return (
            <div>
                <SearchBar onSubmit={this.onSearchSubmit}/> 
                <ImageList />
            </div>
        );
    }
}
export default App;

import React from 'react';
class SearchBar extends React.component{
    state = {userInput:""} // -- (3)
    
    onFormSubmit = (e)=>{
        e.preventDefault(); 
        this.props.onSubmit(this.state.userInput); // -- (4)
    }
    
    render(){ 
        return( 
            <div>
                <form onSubmit={this.onFormSubmit}> 
                    <input 
                        type="text" 
                        onChange={
                            (e)=>{
                                this.setState({userInput:e.target.value}) // -- (3)
                            }
                        }/> 
                </form>
            </div>
        );
    }
}
```


- (3) Let's talk about **state** here. **State** is a plain javascript object that is managed **within a component** to represent changes in the component.
      <br>
      For the component SearchBar, the information of interest is 'input'. It has to be read, and it has to be delivered.
      onChange() event is called whenever a change is made within the specified html tag. You can retrieve the actual input value with **e.target.value**.
      <br>
      By calling **setState({key:value})** , our desired state is updated to the input value of user. And we can feed this state right into the tossed down function **onSubmit**.

- (4) The props from class component is called by using **this**. So once this function is called, **onSearchSubmit** defined        in <App /> will be called and execute the API call we want to make. 

Okay, so this was it for the SearchBar component. In the next post, I will explain about API calls. I will go over the concept of Async & Sync as well. 