# Bootstrap learning notes
Following course https://www.udemy.com/course/the-complete-web-developer-zero-to-mastery/


Getting started - see https://getbootstrap.com/docs/4.4/getting-started/introduction/
``` html
<head>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
</head>
```
**This would load it at the start (when a user opens your webpage)**

Some common bootstrap patterns:

* Navbar - dynamic navigation bar that changes with page size
* Jumbotron - a large area to draw extra attention to something, normall some 'hero' size text.
* Modal - a modal is a dialog box/ pop up that appears at the top of the page.

## Grids

This is one the most popular bootstrap features that solved the problem of  
how difficult it is to place things in HTML/CSS.

``` html
<div class="container">
    <div class="row">
        <div class="col col-sm-6 col-md-12">
        Text here
        </div>
        <div class="col col-sm-3, col-md-6">
        </div>
    </div>
</div>
```
The columns and rows have sizes defined by the [col/row]-[size category]-[size] format. 
 
The [size] is out of 12 blocks. 

col-sm-[x] - size when page size is small 

col-md-[x] - size when page size is medium 

Multiple of these can be stacked e.g. xs, lg, xlg

## Meta tags

These tags are very frequently added at the top of webpages.
 
Bootstrap is developed mobile first, and this tag ensures proper rendeting.

``` html
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
```

## More bootstrap notes

``` html
<p class="text-uppercase"> Some text </p>
<!-- Bootstrap uppercase text function -->

<button class="btn btn-primary btn-xl">
<!-- Bootstrap formatted button -->

```

## Bootstrap layout

*HEALTH WARNING: With modern day web development this is largely superceded by CSS grid.*
 
*CSS grid is much easier to use.*

``` html
<div class="container d-flex align-items-center h-100">
    <!-- h-100 means the container will span the full page -->
    <div class="row">
        <header class="text-center col-12"> 
            <h1 class="text-uppercase">TITLE</header>
        </header>
        <div class="buffer" col-12></div>
        <!-- create a buffer space -->
        <section class="text-center col-12">
            Some stuff
        </section>
    </div>
</div>
```

## 