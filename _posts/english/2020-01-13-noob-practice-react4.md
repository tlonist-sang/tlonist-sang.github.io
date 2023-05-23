---
layout: post
title:  "React.js - simple app practice (4/4)"
subtitle: "Image fetching app, part4 - little more props and css"
description:
date:   2020-01-13 00:45:22 +0900
comments: true
image: /assets/img/shiva_statue.jpeg
optimized_image: /assets/img/shiva_statue.jpeg
category: javascript
tags: JavaScript, React.js
author: tlonist
---

Okay, so the final posting of this simple app. Considering the simplicity of the app, this shouldn't have taken so many postings, but anyways here goes the finale!

[![img]({{ "/assets/img/react2-suggestion1.png"|absolute_url}})]({{ "/assets/img/react2-suggestion1.png"|absolute_url}})

Using API calls to fetch images was what was done previously, and what is left is to pass down the fetched images to the children components for displaying those images.

(again, full source is availbe at [stephen grider's source - pics app](https://github.com/StephenGrider/redux-code/tree/master/pics))
```javascript
class App extends React.Component{
    state = {searchedImages:[]} //state is defined in the beginning --(1)
    onSearchSubmit = async (userInput) =>{
        const response = await unSplashAPI.get('/search/photos',{
            params: {query: userInput},
        })
        this.setState({searchedImages:response.data.results}); //state is 'set' --(2)
    }
    render(){
        return (
            <div className="ui container" style={{marginTop: '10px'}}>
                <SearchBar onSubmit={this.onSearchSubmit}/>
                <ImageList images={this.state.searchedImages}/> 
            </div>
        ); // this.state.searchedImages are passed down to ImageList components --(3)
    }
}
```

(1) Firstly the state to be saved and updated is defined in the beginning, with the name 'searchedImages', which seems quite appropriate.

(2) Then, I set the state by using the React-inherent function **setState**. The state is guaranteed to be updated because I used async-await syntax to wait for images to be fetched from the API server.

(3) From the render() part, the searchedImages state is passed down to ImageList component with the name of images. Now, what should be done in the ImageList components? Remember, I defined another component called ImageCard.js, to specifically describe one Image out of all the retrieved images; and ImageList is just a frame which hordes many individual ImageCard components. Hence, ***ImageList should have lots of ImageCard components, while passing down single image to each one of them.***

I will make ImageList.js as a function-based component, since according to the requirement it doesn't seem to necessitate much pre or post processing.

```javascript

ImageList = (props)=>{
    var images = props.images.map((image)=>{  //--(1)
        console.log(image);
        return <ImageCard image={image} key={image.id}/>
    });

    return (
        <div>{images}</div>
    )
}

export default ImageList;
```

[![img]({{ "/assets/img/react4-images.png"|absolute_url}})]({{ "/assets/img/react4-images.png"|absolute_url}})
By analyzing what is really inside of each individual images, you can figure out how what to pass down to ImageCard.js component and how the component will display actual images. 

First of all I would need the actual image itself from **urls**, of regular size. Next it will be good to add a unique identifier, a **key** to distinguish them from others. If I don't include the **key={image.id}** part, this pops up.

[![img]({{ "/assets/img/react4-warning.png"|absolute_url}})]({{ "/assets/img/react4-warning.png"|absolute_url}})
Having a unique identifier for this kind of repeated element is important, even if the element seems to be unique without the identifier. This is simply because (1) there may come an occasion in the future where the element is no longer unique thus can't be singled out, and (2) it enhances the cosistency in coding - you can always expect to have an identifier.

Now for ImageCard.js
```javascript
import React from 'react';

class ImageCard extends React.Component{
    constructor(props){
        super(props);
        this.imageRef = React.createRef(); // -- (1)
    }
    render(){
        const {description, urls} = this.props.image; // -- (2)
        return(
            <div>
                <img 
                    ref={this.imageRef}
                    alt={description}
                    src={urls.regular}
                />
            </div>
        );
    }
}
export default ImageCard;
```
ImageCard component seems plain. It makes use of the props to make a div element with image tag inside. 

(1) createRef() was used by the author for special purpose. If you just execute the code above, it looks quite ugly, with images of varying sizes occupying large area without order and proper spacing. So he wanted to make it into a grid structure like below.

[![img]({{ "/assets/img/react4-summer.png"|absolute_url}})]({{ "/assets/img/react4-summer.png"|absolute_url}})

In order to do so, computations had to be made specifically for individual images. Especially when it is **loaded**.
[createRef](https://reactjs.org/docs/refs-and-the-dom.html) provides a way to manipulate a DOM (or React) element created inside of the render method. By calling the refererenced object, you can execute some related functions, namely setting up a style for individual image. I will go back after briefly mentioning what was about (2)

(2) is an example of ES6 syntax **destructuring**. I'm quite new to this way of coding, so looked up the [link](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment). There is nothing hard about it but awesomeness. It greatly simplifies the code. 

Okay, going back to (1), the final code looks like
```javascript
class ImageCard extends React.Component{
    constructor(props){
        super(props);
        this.state = {spans:0};
        this.imageRef = React.createRef();
    }
    componentDidMount(){
        this.imageRef.current.addEventListener('load', this.setSpans); // --(1) 
    }
    setSpans = () => { // --(2)
        const height = this.imageRef.current.clientHeight;
        const spans = Math.ceil(height/10);
        this.setState({spans});
    }
    render(){
        const {description, urls} = this.props.image;
        let styles={
            width: '250px'
        }
        return(
            <div style={gridRowEnd:`span ${this.state.spans}`}> 
                <img 
                    ref={this.imageRef}
                    alt={description}
                    src={urls.regular}
                    style={styles}
                />
            </div> // --(3)
        );
    }
}
```

(1) componentDidMount() is a function that is called right after the render() function renders DOM element. Depending on the requirement it can perform lots of post-mount actions. In this case, I added an event listener **load** linked to the element. I had a question, why shouldn't 
```javascript
   componentDidMount(){
        this.setSpans();
    }
```
work? Why do I have to add a listener for setting up the state?

This is because there is a difference between **mounting** and **loading** of element.(Please tell me if I'm mistaken) According to the [official document](https://reactjs.org/docs/react-component.html#componentdidmount), componentDidMount means the component is inserted into the tree, and hence does not guarantee a full loading of the element, the image in this case. (it is natural that it takes some amount of time for retrieving images). Hence if **this.setSpans()** is called just right after the component is mounted, the height and spans part will all be 0, because the image is not fully fetched yet, and there is no way of knowing its height! (try it using console.log) Hence it is important to link the created reference to **load** event to make sure it is ready for computation.


After all the setting like above, you will finally have
[![img]({{ "/assets/img/react4-winter.png"|absolute_url}})]({{ "/assets/img/react4-winter.png"|absolute_url}})


In this posting I covered
1. createRef() and componentDidMount() and their usages.
2. micellaneous stuffs I wanted to talk about (wot?)

Okay. So far was the adventure of simple-react-app-copied-from-a-udemy-course-I-took.
I'll post more about my curiosities and resolutions from tomorrow on. Thanks for reading!
