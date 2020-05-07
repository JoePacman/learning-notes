# CSS learning notes

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

There is also the * which is not used very often. It applies to everything (unless cascading  
over rides it)
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
    border: 5px solid purple; /* border*/
    background-image: url(myimage.jpg); /* background image*/
    background-size: cover; /* fit background image */
    cursor:  pointer; /* what your cursor looks like over this element */
    color: #AA4139; /* colour code */
    color: rgb(0, 255, 0); /* RGB colour*/
    color: rgba(0, 255, 0, 0); /* RGB colour. Fouth number is degree of transparency */
}

li {
    list-style: none; /* style of list icons */
    display: block; /* for lists. inline-block makes list items appear in line */
}
```

