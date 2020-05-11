# CSS learning notes
Following course https://www.udemy.com/course/the-complete-web-developer-zero-to-mastery/

## Basic concepts
The basic structure of all CSS in .css files is 

``` css
Selector {
    Property: value;
}

/* e.g. overwriting h2 properties */
h2 {
    color: red;
}

/* more than one element */

h2, p {
    ...
}

/* Only applies to the secondary elements (p's) inside the primary elements (h2's) */
h2 p {
    ...
}

/* Basically the above but h2 has to be the PARENT of p */
h2 > p {
    ...
}
```
which wouldn't apply to:
``` html

<h2>
    <div>
        <p>Text</p>
    </div>
</h2>
```

``` css
/* only applies on hover over */
h2:hover{
    ...
}

/* only applies to the first/ last one come across of this class */
h2:first-child{
    ...
}
h2:last-child{
    ...
}

/* important - overrides cascading (see below). Not recommended! */
p {
    color: pink !important
}
``` 
Linking a HTML file to a CSS file is done via a link tag in head

``` html
<head>
    <title>CSS</title>
    <link rel="stylesheet type="text/css" href="style.css">
</head>
```

and the most common way is defining with a **class** tag, where the selectors begin with a dot:

``` html
<p class="webtext">Lorem Ipsum</p>
```
``` css
.webtext {
    color: red;
}
```
**id** is also very common and can only be used once. The selectors begin with a hash.
``` html
<div id="div1">
    <p class="webtext">Lorem Ipsum</p>
    <p class="webtext">Lorem Ipsum 2</p>
</div>
```
``` css
#div1 {
    background: blue;
}
```

There is also the * which is not used very often. It applies to everything (unless cascading over rides it)
``` css
* {
    text-align: right:
}
```

There are also old methods of defining styles (rarely done):
Doing in line:
``` html
<header style="background-color: green; color: red">
```
Defining with style tags:
``` html
<style>
    li {
        background-color: purple;
        color: white;
    }
</style>
```

CSS **CASCADES** meaning it takes the last input for a specific selector
e.g. here the paragraph would be green

``` css
p {
    color: pink;
}

p {
    color: green;
}
```

Which selector wins out in the cascade depends on:
- Specificity - the more specific a class is to the element. https://specificity.keegan.st/
- Importance - !important tag
- Source order - the order style sheets are read (if multiple are declared in head tag)

## Common CSS properties

```css
example-selector{
    text-align: center; /* alignment */
    text-decoration: underline; /* includes line through and other props */
    text-transform: uppercase; /* includes lower case too and other props */
    line-height: 20px; /* pixels of lines height */
    font-style: italic; /* italics */
    font-weight: bold; /* bold */
    font-size: 80%; /* adjust font size */
    font-family: "Times New Roman", "Georgia"; /* font type. Can have multiple. If the PC doesn't have a font it tries the next */
    border: 5px solid purple; /* border*/
    background-image: url(myimage.jpg); /* background image*/
    background-size: cover; /* fit background image */
    cursor:  pointer; /* what your cursor looks like over this element */
    color: #AA4139; /* colour code */
    color: rgb(0, 255, 0); /* RGB colour*/
    color: rgba(0, 255, 0, 0); /* RGB colour. Fouth number is degree of transparency */

    font-size: 20px; /* 20 pixel size */
    font-size: 5em; /* 5 times the font size of the containing element's font size */
    font-size: 2rem /* size in relation to size of the route element */
}

li {
    list-style: none; /* style of list icons */
    display: block; /* for lists. inline-block makes list items appear in line */
}

img {
    float: right; /* right or left. Floats image in direction, and text wraps around */
    /* generally best to just use with images, has some odd properties */
}

footer {
    clear: both; /* would ignore the float above */
}
```

What if a user doesn't have the font you want on their PC?

Google fonts are frequently used.
Define it in the header:
``` html
<link href="http://fonts.googleapis.com/css?family=Poiret+one" rel="stylesheet">
```
And add to your CSS
``` css
.webtext {
    font-family: 'Poiret One', cursive;
}
```
Note that this slows a website as its waiting on an API call.
 

## The Box Model

Very common properties you will use a lot and can see in developer tools.
### Margin > Border > Padding > Content

``` css
.boxmodel{
    border: 5px solid red;
    display: inline-block;
    padding: 5px 20px 5px 20px; /* top right bottom left */
    margin: 5px; /* space outside of the border. all around */
    margin: 0px 20px 0px 20px; /* top right bottom left */
}
```

## CSS minify

A common technique to increase website responsiveness is to use a minify tool  
which reduces a css file into a single line by removing all whitespace.

## Flexbox

Flexbox is a really useful css toolbox that is commonly used.

See https://css-tricks.com/snippets/css/a-guide-to-flexbox/

To enable:

``` css
.container {
    display:flex; /* this is the command which will import flexbox */
    flex-wrap: wrap;
}

```
A good way to get good at flexbox is this website https://flexboxfroggy.com/


## CSS3

Some cool CSS3 properties:

```css
img {
    transition: al 1s; /* how long before dynamic features enable */
}

img:hover{
    transform: scale(1, 1);
}
```

This is a great website for mastering some of these more dynamic properties:
https://thoughtbot.com/blog/transitions-and-transforms

Note that some properties e.g. transition above may not be supported by all browsers.
This is a great website to check support for different toolboxes: https://caniuse.com/