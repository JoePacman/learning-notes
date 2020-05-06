# HTML learning notes

Following course https://www.udemy.com/course/the-complete-web-developer-zero-to-mastery/

## Key tags to remember
```html
<head>PAGE HEADER</head> - Header

<body>BODY</body> - Body

<p>Lots and lots of text</p> - Paragraph text

<strong>Bold text</strong> - bold

<em>Itallicised text</em> - ittalicise

<li>Ordered list item</li> - Ordered list

<ul>Unordered list item</ul> - Unordered list

<br> - Break. Self closing (no </br>)

<img src="URL/ path to image source" alt="Alternative text for image, e.g. for 
screen reader" width="42" height="42"></img> - Image (of 42 X 42 pixels)

<a href="otherpage.html" target="_blank">Link name</a> - Anchor (link), Path to  
otherpage.html is from the directory of the current HTML file UNLESS the path  
beings http:// or https:// in which case it will take you through the link.
target is optional and "_blank" indicates open in new tab.

<nav></nav> - Nav. Intended to contain a bunch of navigation (<a></a>) links

<form method="POST"> - Form tag. Method attribute is optional. Default is  
    GET in which case form parameters appear in URL (unsecure)

    Input tags (Self closing): 
    <input type="text" name="fieldName" required> - Type text entry. Required  
    is optional. Name is the field name, generally put on all inputs.

    <input type="email"> - Type email. Checks email is as expected on submit.   
    E.g. contains @ symbol.

    <input type="password" minlength="8"> - Type password. Values obscured. 
      minlength optional attribute.

    <input type="date"> - Type date, gives date dropdown.

    <input type="radio"> - Type radio, gives selectable radio.   
    Then multiple radios of same name would only have one selectable.

    <input type="checkbox"> - Type checkbox (used when multiple can be selected)

    <input type="submit" value="Press me!"> - Type submit button. Value is what  
    appears on the button.

    <input type="reset"> - Type reset all form text values.

    Other tags in Form:
    <select multiple>
        <option value="volvo">Volvo</option>
        <option value="honda">Honda</option>
    </select> - Select tag. Dropdown with above options. Multiple attribute is 
     optional, allows multiple to be selected.

</form>

<div></div> - Used to wrap larger elements, e.g. a form and set their look with  
CSS.

<span></span> - Same as div but it is an 'in line' element.
E.g. within a form:
<span>First Name</span><input type="text" name="FirstName" required>


```

## Useful new tags introduced by HTML5

``` html
<header></header> - Page header
<footer></footer> - Page footer

The above help with SEO (Google ranking)

A number of the <input> tags e.g. max, min, pattern, required
```