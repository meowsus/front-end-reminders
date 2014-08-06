# Frequent Front-End FUX-Ups
Issues caught during code reviews and more information on relevant concepts. This document is intended for easy understanding, quick reference, and speedy resolution.


## JavaScript

### Binding and Triggering Events in jQuery
Rather than using jQuery's shortcut methods for binding and triggering events, we use the `$.on()` and `$.trigger()` methods instead. This is for many reasons, but the most prominent is that __on__ binds to any selector, even if it doesn't exist yet, and both of these methods explicitly explain what you're trying to accomplish. 

You're either binding an event __on__ an element or __trigger__ing an event on an element. 

For example:

```JavaScript
var $myAnchor = $('.my-anchor', $scope);

// instead of 
$myAnchor.click(function (e) {
    e.preventDefault();

    window.alert('Clicked!')
});

// use
$myAnchor.on('click', function (e) {
    e.preventDefault();

    window.alert('Clicked!');
});

// and to trigger later, use
$myAnchor.trigger('click');
```

### Variable Hoisting and Scope
Always define and even predefine your variables at the top of your module's functions. This helps lessen confusion and helps reduce sporadically inserted variables as you need them through your code. Furthermore, always using a `var` statement defines scope and keeps variables from floating around in scope they shouldn't be in. For example:

```JavaScript
WEBLINC.myModuleName = (function () { // scoped globally, see below
    'use strict';

    var init = function ($scope) { // scoped to WEBLINC.myModuleName
            var $dependency = $('.element', $scope), // local to init function
                data; // sets localized variable 'placeholder'

            if (_.isEmpty($dependency)) { return; }

            data = $dependency.data(); // still local to init function

            ...

        }

    WEBLINC.modules.onDomReady(init); // references globally scoped object

    return {
        init: init
    };
}());
```

### Variable Naming
Variables that hold jQuery collections should be prefixed with a dollar sign ('$') to allow future developers to know that they're able to use the jQuery API against any of those variable. Regular variables, and even an array of individual jQuery collections, would not have this prefix:

```JavaScript
var $detailsContainer = $('.wl-product-details', $scope),
    myArray = [],
    instance = {
        $scope: $scope,
        $detailsContainer: $detailsContainer,
        $body: $('body', $scope),
        bodyData: $('body', $scope).data()
    };
```


## Markup

### Basic Semantic HTML
Semantics debates can be endless. When two people have the right idea, yet find themselves arguing over the nuance of the concept, one of them is bound to say something to the effect of "now we're just arguing semantics." At that point the argument should be dropped, the people should hug, and go on about their day.

Similar is the world of Semantic HTML. I won't express my views here at this time, as this isn't exactly my soap-box, and arguing semantics is often fruitless. I will point out that there is a nice diagram [here](http://html5doctor.com/downloads/h5d-sectioning-flowchart.pdf), and leave it at that.

_However_, the most basic concept of Semantic HTML should be upheld, and that is "every tag means something."

For example: headings are `H1-6`, paragraph content goes in `p` tags, tabular data in `table`s, lists of information are either `ul`s or `ol`s depending on the type of information going into the list, etc.

This also means some of these elements should not be children of other elements. A heading does not go inside paragraph, tables aren't intended for layouts, paragraphs or divs don't go in list items, etc.

This is important not only for readability, for both developers, screen readers, and search engine spiders, but is also important to help the browser parse your markup correctly. If the markup is not semantic you can easily experience strange anomalies while styling or manipulating the DOM with JavaScript.

#### Helpful links:
  - [Sections and Outlines of an HTML5 Document](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Sections_and_Outlines_of_an_HTML5_document)
  - [HTML5Doctor](http://html5doctor.com)


### Class or ID Declarations in HAML

#### Static classes and IDs
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

If you don't want a `div`, you can explicitly tell HAML what kind of element you want by using the percent sign '%'. For example:

```HAML
%span.wl-icon--share
```

which would compile to

```HTML
<span class="wl-icon--share"></span>
```

#### Dynamically setting these values

If you do need your class applied to an element, based on a condition, use the following notation:

```HAML
%span.wl-icon{ class: 'wl-icon--share' if @product.share_enabled? }
```

which would compile to

```HTML
<span class="wl-icon"></span>
```

if, in this example, the product does not have sharing enabled, or:

```HTML
<span class="wl-icon wl-icon--share"></span>
```

if it does.

Similarly, if a class or ID must have, say, a suffix that will always be defined, you could use string expansion. For example, a product will always have some kind of template, defined as part of the product's object:

```haml
.wl-product-details{ class: "wl-product-details--#{@product.template" }
  ...
```

which would render to:

```html
<div class="wl-product-details wl-product-details--generic">
  ...
</div>
```

in certain cases.

#### Helpful links
  - [HAML Documentation](http://haml.info/docs.html)


### Readability

When working with ActionView Helpers we should be spacing curly brakets accordingly:

```HAML
.my-element{ data: { attribute_name: 'attribute value' } }
  ...
```

Which is compiled to:

```HTML
<div class="my-element" data-attribute-name="attribute value">
    ...
</div>
```



## Stylesheets

### Adjacent Selectors
Adjacent Selectors allow you to select and style a group of similar elements with the exception of the first element. This is useful in many cases, and sometimes requires a shift in the way styling for a group of elements is thought about. For example, you may naturally want to do something like this:

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

but a simpler implementation could be written as follows:

```SCSS
.wl-product-list {
  .product-list-item + .product-list-item {
    // Each item gets a top border, except the first
    border-top: 1px dotted $border-color;
  }
}
```

#### Helpful links:
  - [Adjacent sibling selectors on MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Adjacent_sibling_selectors)
  - [General sibling selectors on MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/General_sibling_selectors)


### Declarative Order for Sass

Adhering to a declarative order for your CSS blocks is important, since it creates continuity between projects.

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
  opacity: 0.75;
  @include respond-to($medium-breakpoint) {
    margin: 0 0 10px;
    padding: 0;
    clear: both;
    border: 0;
  }
}
```


### Margin and Padding

The first time margin and/or padding is applied, it should be defined for all four directions. This sounds vague, but since we style elements or components in multiple locations, it is a bit vague.

So, if you're styling a component and you have to set it's margin and/or padding, fully define those properties. Then if you're trying to override one of the values at the `_views.css.scss` level, override that specific value.

If, at the top of a Sass block, you're freshly applying margin and/or padding, define it fully, and modify it later in the Sass block as needed.

Here are some examples:

```SCSS
form {
  display: block;
  padding: 20px 10px;

  @include respond-to($medium-breakpoint) {
    padding-left: 0; // override left padding for this state
  }

  @include respond-to($wide-breakpoint) {
    padding-bottom: 10px; // override bottom padding for this state
  }
}
```

```SCSS
// in _components.css.scss
@mixin wl-box {
  margin: 0 0 15px;
  padding: 15px;
  ...
}

// in _views.css.scss
.wl-account-dashboard-view {
  .wl-box {
    margin-bottom: 20px; // overrides one direction
    padding: 10px; // overrides all directions
  }
}

If, of course, you're overriding more than one value later in the block, it might make more sense to redefine each direction. That should be evaluated on a case by case basis.


## Single-Line CSS Declarations

You may be tempted to write a single CSS property / value declaration all in one line, like so:

```SCSS
.wl-property, .wl-property--row { font-weight: bold; }
```

but this opens room for more merge conflicts. Since all of our SCSS is eventually compiled onto one line, we can avoid the potentiality of more conflicts by breaking this out to read:

```SCSS
.wl-property,
.wl-property--row {
  font-weight: bold;
}
```

If every block adheres to this rule, git is able to resolve many more conflicts by itself and better understand what developers are trying to do with each SCSS commit.


## Styling "States" in SCSS

We style elements all the time. When we start to style an element, you can think of it like you're styling an element's default state. Another obvious state of that element might be a `:hover` state. For a link, you'd style the `a` according to how it should look by default, then add a nested `&:hover {}` inside of the parent block to define it's hover state. 

You can think of `@respond-to` in the same way.

For example, here is an element's default state:

```SCSS
.wl-mobile-style-nav {
  display: block;
}
```

Chilren of this parent element also have default states, like so:

```SCSS
.wl-mobile-style-nav {
  display: none;

  a {
    display: block;
    padding: 12px 15px;
    text-decoration: none;
  }
}
```

Next we want to define states. We want to (a) show everything only at the medium breakpoint, (b) put an underline under anchor tags when they're hovered, and (c) turn the achors red at the medium breakpoint:

```SCSS
// Good Example
.wl-mobile-style-nav {
  display: none;
  @include respond-to($medium-breakpoint) {
    display: block;
  }

  a {
    display: block;
    padding: 12px 15px;
    text-decoration: none;
    &:hover {
      text-decoration: underline;
    }
    @include respond-to($medium-breakpoint) {
      color: $medium-anchor-color; // #ff0000
    }
  }
}
```

States should be defined so that they can be easily identified. This means that you want to avoid these two types of common anti-patterns:

```SCSS
// Bad Example
.wl-mobile-style-nav {
  display: none;

  a {
    display: block;
    padding: 12px 15px;
    text-decoration: none;
    &:hover {
      text-decoration: underline;
    }
  }

  // having this state at the bottom of the block works, but it is unclear that
  // we're acting on the code for the default state of .wl-mobile-style-nav.
  // It should be moved up under the second line to be more clear.
  @include respond-to($medium-breakpoint) {
    display: block;
    // just because we've opened a medium breakpoint block doesn't mean we
    // should jam more elements inside of it than are necessary. We want to
    // define a state WITHIN it's element.
    a {
      color: $medium-anchor-color;
    }
  }
}
```

The 'Good Example' (above) is an good example of how the structure should be maintained, for reasons of code clarity.


## Using line-height as a Vertical Alignment Hack

Arguably, anytime you're increasing the `line-height` on an element to a value greater-than one-and-a-half times it's natural height, you're probably trying to do something like vertically align the text inside it's container.

This isn't always the case of course, but a good rule of thumb is to rethink what you're doing if you start to drastically increase an element's `line-height`.

An example where this is commonly seen is inside forms. Most front-end developers want their labels to align, vertically, with their associated input fields when the label/input combinations are supposed to be presented in rows. Let's say the input is 30px tall. If you match the `line-height` of your label to be 30px as well, it looks great... until that label becomes long enough to wrap. Then you have a very, very confusing looking form on your hands.

A better way to handle this is by making both the label and the input field `display: inline-block` with a `vertical-align: middle` and a reasonable `line-height` applied to the label. This, however, opens the question of "what to do about rows that contain textareas". If having a label middle-aligned with a large input element, like a textarea, then I'd specifically code for that type of condition, or, instead of the aforementioned solution, add at least a top padding to each of your labels, so that each label associated with a text box or select element _appears_ vertically centered, until you get to the text-area.

Luckily there are many components in the WebLinc platform that help with form handling, most notable in this case is [wl-property--row](http://demo.weblinc.com/style_guide#wl-property--row).

Just try to rethink what you're doing if you catch yourself considering using `line-height` for some purpose other than strictly for the height between lines of text.


### Z-Index

#### Do you need it?
`z-index` should only be applied as needed. Often I will see a `z-index` applied to try to solve a layering problem that can be solved more appropriately by reordering some markup. Keep in mind that the order that you declare your elements in a block of markup also dictates implicit element stacking rules. Consider this styling:

```SCSS
.example {
  position: relative;
  p {
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

When rendered, the page would show only one block, and that block would read "Five".

This is because each element in markup is automatically considered "above" the element that precedes it. So the last sibling, in this example, will _naturally_ be in the forefront.

Keep this in mind before deciding to use `z-index`.

#### Other Reminders
  - `z-index` is married to `position`. `z-index` will not work unless a `position` has been defined.
  - It's conventional to increment each applied `z-index` by values of 10.
  - Applying `z-index` to a parent also affects it's children. Before you hunt for the exact element you want to appear above another element or section ask yourself, "does it make more sense for this element's container to be `z-index`ed instead?"

#### Useful commenting
It's safe to assume that every time you add a `z-index` property to a style block it's purposeful. You're saying that you want that element to appear over another element and/or under another.

This is why we should always indicate this intent with a comment following the application of this property, clearly explaining our intent. This helps validate the decision and help other developers understand what is happening at a glance. For example:

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

In this block, any other developer can clearly see the intent: the previous developer wants `.wl-page-header` to appear above both `.wl-page-content` and`.wl-page-footer`. So an assumption can be made that, in this example, `.wl-page-content` and `.wl-page-footer` must have `z-index`es set _lower_ than the value applied to `.wl-page-header`.

#### Helpful links:
  - [z-index on MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/z-index)
  - [Understanding CSS z-index on MDN](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Understanding_z_index)
  - [What No One Told You About z-index](http://philipwalton.com/articles/what-no-one-told-you-about-z-index/)










# Rails Base Concepts

## Data Directory

### What is it?
The ```data/``` directory is in the root of every project repository. Its sole purpose is to take it's contents and dump them into the local database during a database seed operation. Adding or modifying the files in this directory will populate Shared Content or Page content inside the of the admin. It purely populates this content in each development environment, locally for each developer to view.

### What to do next?
Since clients can modify the staging and production environments, we rarely reseed these environments. Conversely, during development, we frequently reseed our local environment. So, the rule of thumb is, _every time you add or modify files in the data directory, you must manually update the admin for at least the staging environment afterwards_.

## JavaScripts

### Modules

#### Framework Documentation + Module Structure
Documentation on how to structure your modules can be found on the [WebLinc MODCON Framework Repository Page](http://github.com/weblinc/modcon). "MODCON" is the terrible name I decided to release the WebLinc Modular Framework & Execution Controller under. "WLMFEC" didn't have quite the right ring to it.

#### Functions
Each function in your should perform one, and only one, task as made evident by the name of the function itself. If you think your function name is _too_ descriptive, and long, you're probably on the right track.

By compartmentalizing all of your work into small, manageable functions helps in many ways.

Notably, many of the functions can be "publicized" via the object returned at the end of the module. Consider the `init` function being returned in every module. This allows each module to be handled by the MODCON framework and it's functionality executed on demand.

If you stick to this rule you'll find that if each task has it's own function, you can easily break your solution to a problem down into it's basic pieces, which even aids in the way you start to think about how to solve future problems.

#### Tying into MODCON
Most modules you will write will need to be run, at the very least, when the document is ready. Some modules you will write will need to be rerun when another module tells it that it might need to be rerun. An example of the latter is the vast majority of the functionality on the Product Details page.

Consider the following: a user is on a Product Details page. In the aside there are related products, each with their own Quick View buttons. The page is loaded, so all of the scripts relevant to that page have already run, _but_ if a Quick View button for another product is clicked, those scripts will need to be _rerun_. We handle the first case by adding:

`WEBLINC.modules.onDomReady(init)`

and the second by adding, if necessary,

`WEBLINC.modules.onDomUpdate(init)`

to the bottom of the module, beneath the close of the `init` function and above the `return` block.

This registers your module with the `onDomReady` and `onDomUpdate` queues, native to MODCON, which are fired by the `domReady` and `domUpdate` functions.

This is is where scope comes into play...

#### Scope is everything
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


## Stylesheets

### Components

#### When to scope view styles to a component
If you have a piece of code that requires styling and is referenced across multiple views, or between a layout and a view, this is usually a good indicator that a component is needed.

An exception to this rule is with entire view style blocks. The best example is `.wl-category-browse-view` vs `.wl-search-results-view`. These are both views, and their styling is specific to the contents of the view, but each of these pages base styles are almost identical.

Combining these styles in a chain, like so:

```SCSS
// ===============================================
// wl-category-browse-view and wl-search-results-view shared styles
// ===============================================

.wl-category-browse-view, .wl-search-results-view  {

  h1 {
    @extend %visually-hidden;
  }

  // top controls
  .wl-browsing-controls--top {
    padding: 0 0 10px;
    border-bottom: 1px solid $border-color;
  }

  // bottom controls
  .wl-browsing-controls--bottom {
    padding: 10px 0 0;
    border-top: 1px solid $border-color;
  }

  ...

}
```

is problematic because, if in the future new functionality specific to one of these views is introduced, neither of these views are unique. You'd have to re-separate out these views to make unique changes to one or the other, or you'll keep this block and just add another block underneath it that has specific styles only for the one view and not the other.

A better solution would be as follows, keeping in mind this solution is for views that are strikingly similar, such as the views in the example:

```SCSS
// ===============================================
// wl-category-browse-view
// ===============================================

.wl-category-browse-view {

  h1 {
    @extend %visually-hidden;
  }

  // top controls
  .wl-browsing-controls--top {
    padding: 0 0 10px;
    border-bottom: 1px solid $border-color;
  }

  // bottom controls
  .wl-browsing-controls--bottom {
    padding: 10px 0 0;
    border-top: 1px solid $border-color;
  }

  ...

}

// ===============================================
// wl-search-results-view
// ===============================================

.wl-search-results-view {
  @extend .wl-category-browse-view;
}
```

This way the base styles for `.wl-search-results-view` are inherited from `.wl-category-browse-view`, and can be easily added to for elements that are specifically different insie `.wl-search-results-view`.


#### Structure of a component
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

### Variables

#### Color Variables: Descriptive vs. Functional

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
