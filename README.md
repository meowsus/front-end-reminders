# Work Blunders

A Living Document Cataloging Frequently Made Front-end Mistakes at Work

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

## Declarative Order for CSS

Each CSS block must follow this order:
  1. Extending + Including (except for respond-to)
  2. Display + Positioning
  3. Margins + Padding
  4. Width + Height
  5. Floating + Clearing
  6. Font Styling
  7. Backgrounds + Borders
  8. Everything else

For example:

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
  background-color: $section-background-color;
  border: 1px solid $section-border-color;
  @include respond-to($medium-breakpoint) {
    margin: 0 0 10px;
    padding: 0;
    clear: both;
    border: 0;
  }
}
```


## Margin and Padding

At the start of each block specify each directional value explicitly, as needed. Then, later in the block, if you need to override _one_ of the values, override that specific value, like so:

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


## Naming Conventions

### Filenames
Filenames must utilize underscores, not dashes
  - __Right:__ `my_excellent_module.js`
  - __Wrong:__ `my-excellent-module.js`

### JavaScript Modules
JavaScript modules should be named (in camelCase) according to their functionality, and their filenames (in snake_case) should match:

This function:
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

... should be named `my_excellent_module.js`


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
  3. a module in the app's asset pipeline
