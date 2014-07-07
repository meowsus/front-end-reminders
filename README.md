# Frequent Front-End FUX-Ups

Little issues frequently caught during code reviews, for easy and quick future reference and resolution.



## Adjacent Selectors in CSS
This type of pattern is frequently seen in reviews:
```SCSS
.wl-product-list {
  .product-list-item {
    // Each item gets a top border
    border-top: 1px dotted $border-color;
    &:first-child {
      // Except the first
      border-top: 0;
    }
  }
}
```

A more sustainable way to write this would be:
```SCSS
.wl-product-list {
  .product-list-item + .product-list-item {
    // Each item gets a top border, except the first
    border-top: 1px dotted $border-color;
  }
}
```

More info on less than straightforward selectors:
  - [Adjacent sibling selectors on MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Adjacent_sibling_selectors)
  - [General sibling selectors on MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/General_sibling_selectors)



## Applying Z-Indexes in CSS

### Do you need it?
`z-index`es should only be applied when they need to be. Keep in mind that the order that you declare your elements in a block of markup also dictates implicit element stacking rules. Consider this styling:

```SCSS
.example {
  position: relative;
  > p {
    display: block;
    position: absolute;
    top: 0;
    width: 100%;
    height: 20px;
    color: $example-color;
    background-color: $example-bg-color;
    outline: 1px solid $example-outline-color;
  }
}
```

As it's applied to the following markup:

```HAML
.example
  %p One
  %p Two
  %p Three
  %p Four
  %p Five
```

It would appear as though only one block was visible, and that block would read "Five".

This is because each element in markup is automatically considered "above" the element that precedes it. So the last sibling, in this example, will naturally be in the forefront.

Keep this in mind before deciding to use `z-index`.

### Useful commenting
It's safe to assume that every time you add a `z-index` property to a style block it's purposeful. You want that element to appear over another element and/or under another.

This is why we should always indicate this with a comment following the application of this property, for example:

```SCSS
.wl-page-header {
  @extend %cf;
  position: relative;
  z-index: 30; // over wl-page-content and wl-page-footer
  margin: 0 0 15px;
  background: $background-color;

  ...

}
```

In this block, any other developer can clearly see the intent: you want `.wl-page-header` to appear overtop both `.wl-page-content` and`.wl-page-footer`. So an assumption can be made that, in this example, `.wl-page-content` and `.wl-page-footer` must have `z-index`es set _lower_ than the value applied to `.wl-page-header`.

More information on z-index:
  - [z-index on MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/z-index)


## Basic Semantic HTML

Every developer, for every project, should always make sure their markup is semantic. At the lowest level this means, in part, that tags have meanings and those meanings should be upheld.

For example: headings are `H1-6`, paragraph content goes in `p` tags, tabular data in `table`s, lists of information are either ordered or unordered lists, etc.

Furthermore, some of these elements should not be children of other elements. A heading does not go in a paragraph, tables aren't intended for layouts, paragraphs don't go in list items, etc.

This is important not only for readability, for both developers, screen readers, and search engine spiders, but is also important to help the browser parse your markup correctly. If the markup is not semantic you can easily experience strange anomalies while styling or manipulating the DOM with JavaScript.



## Class or ID declarations in HAML

HAML is smart. By default, without explicitly specifying an element via HAML's class (the dot '.') and/or ID (the octothorpe, or hash '#') syntax only, HAML assumes you want to use `div`, like so:

```HAML
#my-id.my-class
  Hey There
```

compiles to:

```HTML
<div id="my-id" class="my-class">
    Hey There
</div>
```

If you don't want a `div`, you can explicitly tell HAML what kind of element you want:

```HAML
%span.wl-icon--share
```

which would compile to

```HTML
<span class="wl-icon--share"></span>
```

__*This is the appropriate way to define classes that don't need to be dynamic in nature.*__

If you do need your class applied to an element, based on a condition, use the following notation:

```HAML
%span.wl-icon{ class: 'wl-icon--share' if @product.share_enabled? }
```

which would compile to

```HTML
<span class="wl-icon"></span>
```

if, in this example, the product does not have sharing enabled, and:

```HTML
<span class="wl-icon wl-icon--share"></span>
```

if it does.



## Color Variables: Descriptive vs. Functional

When adding a new color to be used across the site, we add it to the top of `_vars.css.scss`, inside the _Descriptive Names_ section, like so:

```SCSS
$gold:                        #bb9a53;
$white:                       #ffffff;
$black:                       #000000;
$almost-black:                #222222;
$black-alpha-75:              rgba($black, 0.75);
```

This way we can use each Descriptive Name variable, inside the _Functional Names_ section. Each of the Functional Name variables are unique to a use case throughout the site. For example, if you're trying to style the entire footer, you would add a block to your `_vars.css.scss` file, like so:

```SCSS
$footer-color:                $white;
$footer-bg-color:             $almost-black;
$footer-heart-color:          $gold;
$footer-form-color:           $white;
$footer-form-bg-color:        $black-alpha-75;
$footer-form-envelope-color:  $gold;
```

Then, your actual `page_layout.css.scss` file, where the styling should take place for the page's footer, would reference the Functional Name variables _only_:

```SCSS
.wl-footer {
  display: block;
  color: $footer-color;
  background-color: $footer-bg-color;
  form {
    @extend %wl-inline-form;
    color: $footer-form-color;
    background-color: $footer-form-bg-color;
    .wl-icon {
      width: 20px;
      height: 20px;
      font-size: 20px;
      color: $footer-form-envelope-color;
    }
  }
}
```

With this pattern we infrequently add new Descriptive Name variables and very frequently add Functional Name variables, allowing us to make massive color changes to the site by only ever editing one file, the `_vars.css.scss` file.



## Components in CSS

### Structure
Each and every component and modifier should be grouped and referenced in triplicate inside the `components.css.scss` file. One of the references is a mixin for including, one is a placeholder selector for extending, and one is a class selector for direct class reference. Here is an example of `wl-button` and `wl-button--small`:

```SCSS
// ===============================================
// wl-button (for @include)
// ===============================================

@mixin wl-button {
  @include transition(background, 0.2s, ease);
  display: inline-block;
  margin: 0;
  padding: 11px 20px;
  text-align: center;
  line-height: 1;
  font-size: 22px;
  font-family: $tertiary-font;
  font-weight: normal;
  text-transform: uppercase;
  text-decoration: none;
  color: $button-color;
  background: $button-bg-color;
  border: 0;
  cursor: pointer;
  white-space: nowrap;
  &:hover {
    color: $button-color;
    text-decoration: none;
    background: $button-hover-bg-color;
  }
}

// ===============================================
// wl-button (for @extend)
// ===============================================

%wl-button {
  @include wl-button;
}

// ===============================================
// wl-button (class selector)
// ===============================================

.wl-button {
  @extend %wl-button;
}

// ===============================================
// wl-button--small (for @include)
// ===============================================

@mixin wl-button--small {
  @include wl-button;
  padding: 7px 10px;
  font-size: 16px;
}

// ===============================================
// wl-button--small (for @extend)
// ===============================================

%wl-button--small {
  @include wl-button--small;
}

// ===============================================
// wl-button--small (class selector)
// ===============================================

.wl-button--small {
  @extend %wl-button--small;
}
```



## Declarative Order for CSS

Each CSS block must follow this order:
  1. Extending + Including (but not respond-to)
  2. Display + Positioning
  3. Margins + Padding
  4. Width + Height
  5. Floating + Clearing
  6. Font Styling
  7. Backgrounds + Borders
  8. Everything else
  9. @include respond-to();

A contrived example:
```SCSS
.section {
  @include box-sizing(border-box);
  display: block;
  position: relative;
  z-index: 10; // see what I did there?
  margin: 10px;
  padding: 5px;
  width: 100%;
  min-height: 50px;
  float: left;
  color: $section-color;
  text-transform: uppercase;
  border: 1px solid $section-border-color;
  background-color: $section-background-color;
  outline: 1px solid $section-outline-color;
  @include respond-to($medium-breakpoint) {
    margin: 0 0 10px;
    padding: 0;
    clear: both;
    border: 0;
  }
}
```


## Margin and Padding

At the start of each block _specify each directional value_ explicitly, as needed. Then, later in the block, if you need to override _one_ of the values, override that specific value, like so:

```SCSS
form {
  display: block;
  padding: 20px 10px;

  @include respond-to($medium-breakpoint) {
    padding-left: 0;
  }

  @include respond-to($wide-breakpoint) {
    padding-bottom: 10px;
  }
}
```

If, of course, you're overriding more than one value later in the block, it would make more sense to redefine each direction.



## Naming Conventions

### Filenames
Filenames must utilize underscores, not dashes
  - __Right:__ `my_excellent_module.js`
  - __Wrong:__ `my-excellent-module.js`

### JavaScript Modules
JavaScript modules should be named (in camelCase) according to their functionality, and their filenames (in snake_case) should match.

For example, the function:

```JavaScript
WEBLINC.myExcellentModule = (function () {
    'use strict';

    var init = function () {
            window.alert('My Excellent Module is Excellent');
        };

    WEBLINC.modules.onDomReady(init);

    return {
        init: init
    };
}());
```

should be named `my_excellent_module.js`



## Single-Line CSS Declarations

You may be tempted to write a single CSS property / value declaration all in one line, like so:

```SCSS
.wl-property, .wl-property--row { font-weight: bold; }
```

but this opens room for more merge conflicts. Since all of our SCSS is compiled onto one line, we can avoid the potentiality of more conflicts by breaking this out to read:

```SCSS
.wl-property,
.wl-property--row {
  font-weight: bold;
}
```

If every block adheres to this rule, git is able to resolve many more conflicts by itself and better understand what developers are trying to do with each SCSS commit.



## Style Guide

You must add a new section to the style guide each time:
  1. A new color is added to the `_vars.css.scss` file,
  2. A new icon is added to the Icon font family,
  3. A new component is added to the `components.css.scss` file, either by creating a new component or by moving a group of styles out of another SCSS file and into the components file,
  4. A new helper mixin is added to `_helpers.css.scss`,
  5. A new JavaScript module is added to the app's asset pipeline

You must double check the style guide for accuracy to make sure the description and/or visual representation of a section is applicable / correct each time you have modified:
  1. a component in the `components.css.scss` file,
  2. a helper in the `_helpers.css.scss` file,
  3. a JavaScript module in the app's asset pipeline



## Using line-height as an Alignment Hack

Arguably, anytime you're increasing the `line-height` on an element to a value greater-than one-and-a-half times it's natural height, you're probably trying to do something like vertically align the text inside it's container.

This isn't always the case of course, but a good rule of thumb is to rethink what you're doing if you start to drastically increase an element's `line-height`.

An example where this is commonly seen is inside forms. Most front-end developers want their labels to align, vertically, with their associated input fields when the label/input combinations are supposed to be presented in rows. Let's say the input is 30px tall. If you match the `line-height` of your label to be 30px as well, it looks great... until that label becomes long enough to wrap. Then you have a very, very confusing looking form on your hands.

A better way to handle this is by making both the label and the input field `display: inline-block` with a `vertical-align: middle` and a reasonable `line-height` applied to the label. This, however, opens the question of "what to do about rows that contain textareas". If having a label middle-aligned with a large input element, like a textarea, then I'd specifically code for that type of condition, or, instead of the aforementioned solution, add at least a top padding to each of your labels, so that each label associated with a text box or select element _appears_ vertically centered, until you get to the text-area.

Luckily there are many components in the WebLinc platform that help with form handling, most notable in this case is [wl-property--row](http://demo.weblinc.com/style_guide#wl-property--row).

Just try to rethink what you're doing if you catch yourself considering using `line-height` for some purpose other than strictly for the height between lines of text.



# Front-End Rails Knowledge Base for Engines

More information on important base concepts.



## JavaScript Modules

### Framework Documentation + Module Structure
Documentation on how to structure your modules can be found on the [WebLinc MODCON Framework Repository Page](http://github.com/weblinc/modcon). "MODCON" is the terrible name I decided to release the WebLinc Modular Framework & Execution Controller under. "WLMFEC" didn't have quite the right ring to it.

### Functions
Each function in your should perform one, and only one, task as made evident by the name of the function itself. If you think your function name is _too_ descriptive, and long, you're probably on the right track.

By compartmentalizing all of your work into small, manageable functions helps in many ways.

Notably, many of the functions can be "publicized" via the object returned at the end of the module. Consider the `init` function being returned in every module. This allows each module to be handled by the MODCON framework and it's functionality executed on demand.

If you stick to this rule you'll find that if each task has it's own function, you can easily break your solution to a problem down into it's basic pieces, which even aids in the way you start to think about how to solve future problems.

### Tying into MODCON
Most modules you will write will need to be run, at the very least, when the document is ready. Some modules you will write will need to be rerun when another module tells it that it might need to be rerun. An example of the latter is the vast majority of the functionality on the Product Details page.

Consider the following: a user is on a Product Details page. In the aside there are related products, each with their own Quick View buttons. The page is loaded, so all of the scripts relevant to that page have already run, _but_ if a Quick View button for another product is clicked, those scripts will need to be _rerun_. We handle the first case by adding:

`WEBLINC.modules.onDomReady(init)`

and the second by adding, if necessary,

`WEBLINC.modules.onDomUpdate(init)`

to the bottom of the module, beneath the close of the `init` function and above the `return` block.

This registers your module with the `onDomReady` and `onDomUpdate` queues, native to MODCON, which are fired by the `domReady` and `domUpdate` functions.

This is is where scope comes into play...

### Scope is everything
In the previous sections Product Details example, you might be wondering why we we don't run into problems running, essentially, the same JavaScript on the same page two times.

This is because the MODCON framework provides your `init` function with a `$scope` variable, which should be used to select the jQuery collection your module will rely on.

When MODON's `WEBLINC.modules.domReady` function runs your module's `init` function, on page load (which happens automatically), the supplied `$scope` variable it is set to `$(document)`. When `WEBLINC.modules.domUpdate` runs your module's `init` function, when a significant portion of the DOM has changed (which can be manually triggered by any module), the client module that calls the `WEBLINC.modules.domUpdate` function supplies a new scope to the function, which is passed to each module that has been added to the domUpdate queue.

Each module is based on a selectable dependency in the DOM, which should be dependent on the current automatically supplied scope. The job of the `init` function is to ensure a specific jQuery collection exists, immediately exit if it does not, and to fire the necessary functions in the module to allow it to run properly.



## Overriding Assets

### What is overriding?
The process of overriding only applies to files inside the asset pipeline, and only on enginized Rails projects. Via the command line, you can explicitly tell the platform which file you want it to make available to you, for modification. Since the platform is comprised of many gems, theoretically all set to different release versions, it would be a bear to find not only the file you want to edit, but the version of the file that is compatible with the platform in it's current state.

### When to override
You will want to override files when they need modification. A good example are the stylesheets. During a new build, chances are very high that you will need to modify each SCSS file from the __weblinc-ecommerce__ (base) gem.

You might be wondering how anything even works if these files aren't all native to your application. The Rails platform actually drops down to the gems to load the proper assets. Overriding them just makes a copy and puts them in your application, so that you can modify them to suit your needs.

### How to override
In your projects working directory, simply run the following command: `bundle exec weblinc override` to see examples of how to use the override tool.

You essentially need to supply the override command with (a) what type of file you want, and (b) that path to the file that you want. For example:

```bash
bundle exec weblinc override layouts=store_front/application.html.haml
```

would override the __weblinc-ecommerce__ gems' default layout file, which needs to be heavily modified for a new build. This would not only bring in the file, but the version of the file compatible with the version of the ecommerce gem as listed in the projects __Gemfile__.

This override command does not, at this point, differentiate between gems. If you typed, for example:

```bash
bundle exec weblinc override stylesheets=store_front
```

the override program would bring in all stylesheets specific to the store_front (as opposed to the admin), for all gems. You would need to go through each of the overridden files and remove the ones you didn't really want to override with that command. Therefore, granular specificity is preferred. There's more typing, but you ensure that you get just the files you need, and no more.

### When to commit
When ever you run the ```bundle exec weblinc override``` command to pull a asset from any remote engine repository into your local repository, you _must_ commit immediately _after_ the override, and _before_ you begin to adjust the file.

Furthermore, your commit message for the override should be phrased much like "Overrides Product Detail Clothing Template", or something to the effect, having "Overrides" as the first word in the commit message.

This allows a reviewer to clearly see which commits are for overrides and which are for actual work needing review. The difference being that overridden files need no review, since they have been reviewed and accepted previously. If the commits are not separated the reviewer can easily assume that the developer had written the entire file from scratch.



## The Data Directory

### What is it?
The ```data/``` directory is in the root of every project repository. Its sole purpose is to take it's contents and dump them into the local database during a database seed operation. Adding or modifying the files in this directory will populate Shared Content or Page content inside the of the admin. It purely populates this content in each development environment, locally for each developer to view.

### What to do next?
Since clients can modify the staging and production environments, we rarely reseed these environments. Conversely, during development, we frequently reseed our local environment. So, the rule of thumb is, _every time you add or modify files in the data directory, you must manually update the admin for at least the staging environment afterwards_.
