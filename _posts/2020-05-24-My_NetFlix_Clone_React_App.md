---
layout: post
title: My Netflix clone React App - Day1 - Progress - 10%
published: true
disqus_identifier: My Netflix clone React App
comments: true
---

​     

## Idea   

I want to create a clone of Netflix app, at least minimum possibilities.

## Getting Started with React App Development

The first step is to invoke the following command:

```javascript
npx create-react-app netflix-clone
```

This creates a basic react app, prepares the development environment, by setting up folders like: public, src, and setting up files like: package.json, index.html within public folder, index.js within src folder and various other important files.



#### Initial Dependencies for our App

1. React
2. ReactDOM

If you want below screen to appear:

![image-20200524125021937]({{site.url}}/assets/img/netflix_clone/hello.png)

Then you need to basically update 3 files,

1. index.js within src folder
2. index.html within public folder
3. Add a style.css, for controlling font

Within index.js file, you will have the following code:

```javascript
import React from 'react';

import ReactDOM from 'react-dom';



class Main extends React.Component{

render(){

return(<h1>Hello World</h1>);

}



}



// we need to ask ReactDOM to render the above component within 'root' div of index.html

ReactDOM.render(<Main/>, document.getElementById('root'));
```



##### Code Commentary

1. Main - It is a class component. It is extending React.Component. The render() function, whatever is inside this, gets displayed ultimately from this component.
2. ReactDOM is responsible for rendering our Main component, within root div of index.html

Within index.html file, you need to have a div with id='root', within the body, with following code:

```html
<html>

<head>
    <link rel="stylesheet", href="style.css">
</head>

<body>

<div id='root'></div>

</body>

</html>
```



Within style.css file, you need to have a css markup for styling components within body:

```css
body{

font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;

}
```



### Getting content for the movies

```html
<a href="https://developers.themoviedb.org/3/getting-started/introduction">TheMovieDB</a>
```



#### Creating a Production Grade Build

When we are in development mode, the build occurs with the following command:

```
npm start
```



But, when we want to release into production, the code needs to be optimized, this is done with the following command:

```
npm run build
```



### Features going to be dealt with:

1. React Hooks - for state management
2. Fetch API - for retrieving data



#### CSS-Aside

CSS has a concept called ***Box Model*** where in we treat every HTML element like a box.

There are 4 things within each box of HTML element:

1. Content
2. Padding
3. Border
4. Margin

Content is what we actually see, for example: text or image.

( Best place to learn about CSS-BOX is https://www.w3schools.com/css/css_boxmodel.asp )

Around the Content, there is a Padding, this is something like a small area created around the Content.

Then comes Border, Border is again something like a small area around Content & Padding.

Then comes Margin, Margin is like an area created outside the Border.

For example:

```css
div{
width:  200px;
height: 200px;
padding: 20px;
border: 15px solid green;
margin: 20px;
}
```

![image-20200524142018107]({{site.url}}/assets/img/netflix_clone/hello_box.png)

### Current Status of the web app - 10%

1. Added h1 tags, with borders, which simulates boxes, containing movie content.
2. Built app
3. Deploying with Surge

Check out my app ( incomplete ) at https://shiny-plastic.surge.sh/

### Github Project

https://github.com/inspire99/netflix-clone