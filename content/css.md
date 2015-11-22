---
title: "Writing CSS"
---

When you are building a large application, CSS can quickly become a major pain point for every developer on the team. In this article, I want to cover the main issues you will encounter, and our solutions for them. These solutions are not very turn key and will require discipline and energy to maintain, but will save you countless hours or fighting the cascade and slowing your app down with specificity hacks, and `!important` at the end of every line.

### Issues with CSS in modern web apps.

1. Which preprocessor should I use?
2. How do I manage all my CSS files?
3. Specificity and global name space are killing me!
4. What is the best way to organize my CSS inside of the file?
5. CSS is so basic, why should I lint?. 

I want to address each of these issues in this article. Weather your app is large or small, or some where in between, you will run into these issues and have to address them.

## Choosing a preprocessor

Many people make this a big deal, but the reality is, this will not make your CSS amazing and solve all your problems. It can save you time when used right, but it can also ruin your output code and cause a great deal of hassle if used wrong.

There are 4 main preprocessors for `CSS`:

### Sass/Scss

Sass was the first, starting in 2006 within the ruby community. Sass is to CSS as CoffeeScript is to js amd Haml to Html. The ruby train was chugging and gems were the bees knees. The syntax was indentation based and had no `{}` after selectors or media quarries, and no `;` after `property: value` decelerations. Over time a more CSS styled syntax was added with the file extension `.less` instead of `.sass`, and in 2012 `libsass` started. It was the start of a full rewrite of the Sass compiler in `C`, and has now become the standard in the Sass community. Note, Sass is not software as a style sheet, and doesn't cost $9.99 per month.

The main features of Sass are:

#### Variables
Have a color in 4 different files? [Make a var for it](https://www.youtube.com/watch?v=iHmLljk2t8M). Have the same padding inside all your divs with text? [Make a var for it](https://www.youtube.com/watch?v=yYey8ntlK_E).

##### Your Scss file:

```scss
$nav-height: 48px;

.nav {
    background: #4078c0;
    height: $nav-height;
    padding: 0 6px;
}

.link {
    font-size: 12px;
    // Get the text vertically centered in the nav.
    line-height: $nav-height;
    color: #ccc;
}

.link:hover {
    color: #fff;
}

```

###### Your output:

```css
.nav {
    background: #4078c0;
    height: 48px;  
    padding: 0 6px;
}

.link {
    font-size: 12px;
    line-height: 48px;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```

#### PreProcessed imports
Split your Scss into many files, and import them into one file. Sass will then compile them into a single files so there is only one http request for your `css`.

##### Your scss files:

###### variables.scss
```scss
$nav-height: 48px;
```

###### nav.scss
```scss
.nav {
    background: #4078c0;
    height: $nav-height;
    padding: 0 6px;
}
```
 
###### link.scss
```scss
.link {
    font-size: 12px;
    // Get the text vertically centered in the nav.
    line-height: $nav-height;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```

###### index.scss
```scss
@import 'variables.scss';
@import 'nav.scss';
@import 'link.scss';
```
 
##### Your output:
 
```css
.nav {
    background: #4078c0;
    height: 48px;  
    padding: 0 6px;
}

.link {
    font-size: 12px;
    line-height: 48px;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```
 
#### Mixins
Predefined set of styles that can take arguments that adjust it's output. Maybe your headings have the same font-family and font-weight, but have a variable font-size. You could make a mixin that accepts a size argument and the outputs all three properties.

##### Your scss files:

###### variables.scss
```scss
$nav-height: 48px;
```

###### mixins.scss
```scss
@mixin link-style($font-size) {
    font-size: $font-size;
    font-family: sans-serif;
    font-weight: 100;
}
```

###### nav.scss
```scss
.nav {
    background: #4078c0;
    height: $nav-height;
    padding: 0 6px;
}
```
 
###### link.scss
```scss
.link {
    @include link-style(12px);
    // Get the text vertically centered in the nav.
    line-height: $nav-height;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```

###### index.scss
```scss
@import 'variables.scss';
@import 'mixins.scss';
@import 'nav.scss';
@import 'link.scss';
```
 
##### Your output:
 
```css
.nav {
    background: #4078c0;
    height: 48px;  
    padding: 0 6px;
}

.link {
    font-size: 12px;
    font-family: sans-serif;
    font-weight: 100;
    line-height: 48px;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```
 
#### Functions
Functions are smart values for properties. A really common usecase is converting a px value to the correct rem value, eg `font-size: rem(14px);`.

##### Your scss files:

###### variables.scss
```scss
// Setting $root-font-size to 100% so that the $rem-base is 16px.
$root-font-size : 100%;
$rem-base       : 16px;
$nav-height     : 48px;
```

###### functions.scss
```scss
// Strip Unit
// Removes the unit (e.g. px, em, rem) from a value, returning the number only.
// @param {number} $num - Number to strip unit from.
// @return The same number, sans unit.
@function strip-unit($num) {
  @return $num / ($num * 0 + 1);
}


// Convert to Rem
// Converts a pixel value to matching rem value. *Any* value passed, regardless of unit, is assumed to be a pixel value. By default, the base pixel value used to calculate the rem value is taken from the `$rem-base` variable.
// @param {number} $value - Pixel value to convert.
// @return A number in rems, calculated based on the given value and the base pixel value.
@function rem($value, $base-value: $rem-base) {
  $value: strip-unit($value) / strip-unit($base-value) * 1rem;
  @if ($value == '0rem') { $value: 0; } // Turn 0rem into 0
  @return $value;
}

// Both off these functions come from foundation for apps: https://github.com/zurb/foundation-apps/blob/master/scss/helpers/_functions.scss
```

###### mixins.scss
```scss
@mixin link-style($font-size) {
    font-size: rem($font-size);
    font-family: sans-serif;
    font-weight: 100;
}
```

###### elements.scss
```scss
body {
    font-size: $root-font-size;
}
```

###### nav.scss
```scss
.nav {
    background: #4078c0;
    height: $nav-height;
    padding: 0 rem(6px);
}
```
 
###### link.scss
```scss
.link {
    @include link-style(12px);
    // Get the text vertically centered in the nav.
    line-height: $nav-height;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```

###### index.scss
```scss
@import 'variables.scss';
@import 'functions.scss';
@import 'mixins.scss';
@import 'elements.scss';
@import 'nav.scss';
@import 'link.scss';
```
 
##### Your output:
 
```css
body {
    font-size: 100%;
}

.nav {
    background: #4078c0;
    height: 3rem;  
    padding: 0 .375rem;
}

.link {
    font-size: .75rem;
    font-family: sans-serif;
    font-weight: 100;
    line-height: 3rem;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```

#### Nesting
Probably the most widely used feature of all preprocessors, and also the most deadly feature. Nesting lets you write a new selector with properties and all inside of another selector. This would then our put a nested selector.

##### Your scss files:

###### variables.scss
```scss
// Setting $root-font-size to 100% so that the $rem-base is 16px.
$root-font-size : 100%;
$rem-base       : 16px;
$nav-height     : 48px;
```

###### functions.scss
```scss
// Strip Unit
// Removes the unit (e.g. px, em, rem) from a value, returning the number only.
// @param {number} $num - Number to strip unit from.
// @return The same number, sans unit.
@function strip-unit($num) {
  @return $num / ($num * 0 + 1);
}


// Convert to Rem
// Converts a pixel value to matching rem value. *Any* value passed, regardless of unit, is assumed to be a pixel value. By default, the base pixel value used to calculate the rem value is taken from the `$rem-base` variable.
// @param {number} $value - Pixel value to convert.
// @return A number in rems, calculated based on the given value and the base pixel value.
@function rem($value, $base-value: $rem-base) {
  $value: strip-unit($value) / strip-unit($base-value) * 1rem;
  @if ($value == '0rem') { $value: 0; } // Turn 0rem into 0
  @return $value;
}

// Both off these functions come from foundation for apps: https://github.com/zurb/foundation-apps/blob/master/scss/helpers/_functions.scss
```

###### mixins.scss
```scss
@mixin link-style($font-size) {
    font-size: rem($font-size);
    font-family: sans-serif;
    font-weight: 100;
}
```

###### elements.scss
```scss
body {
    font-size: $root-font-size;
}
```

###### nav.scss
```scss
.nav {
    background: #4078c0;
    height: $nav-height;
    padding: 0 rem(6px);
    
    .logo {
        background-image: src('logo.png');
        background-size: contain;
        background-repeat: no-repeat;
        background-position: center;
    
        height: rem(24px);
        width: rem(62px);
    }
}
```
 
###### link.scss
```scss
.link {
    @include link-style(12px);
    // Get the text vertically centered in the nav.
    line-height: $nav-height;
    color: #ccc;
    
    &:hover {
        color: #fff;
    }
}
```

###### index.scss
```scss
@import 'variables.scss';
@import 'functions.scss';
@import 'mixins.scss';
@import 'elements.scss';
@import 'nav.scss';
@import 'link.scss';
```
 
##### Your output:
 
```css
body {
    font-size: 100%;
}

.nav {
    background: #4078c0;
    height: 3rem;  
    padding: 0 .375rem;
}

.nav .logo {
    background-image: src('logo.png');
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;

    height: 1.5rem;
    width: 3.875rem;
}

.link {
    font-size: .75rem;
    font-family: sans-serif;
    font-weight: 100;
    line-height: 3rem;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```




### Less
Less is the middle child of CSS preprocessors. Coming out in 2009, it was made big with Bootstrap. Less tries to take a simpler approach than Sass did, doing away with some of the complexity, and using syntax more closely related to CSS. Less has about half the popularity as Sass, and with version 4 of Bootstrap, it is no longer the preprocessor used to build it. But for many because of Bootstrap, it is the first one they used and is generally considered a better new comers option over the others. Less is written in js, and therefor is faster than the older ruby based Sass, which can see compile times approaching a min in larger projects. Some things to note about Less, there are no true if statements nor are their functions that can be called as values to properties. This can be a deal breaker to many when you start writing your CSS more pragmatically. At the same time without these two features, you do get simpler code on the Less side.
 
The main features of Less are:
#### Variables
Have a color in 4 different files? [Make a var for it](https://www.youtube.com/watch?v=iHmLljk2t8M). Have the same padding inside all your divs with text? [Make a var for it](https://www.youtube.com/watch?v=yYey8ntlK_E).

##### Your Less file:

```less
@nav-height: 48px;

.nav {
    background: #4078c0;
    height: @nav-height;
    padding: 0 6px;
}

.link {
    font-size: 12px;
    // Get the text vertically centered in the nav.
    line-height: @nav-height;
    color: #ccc;
}

.link:hover {
    color: #fff;
}

```

###### Your output:

```css
.nav {
    background: #4078c0;
    height: 48px;  
    padding: 0 6px;
}

.link {
    font-size: 12px;
    line-height: 48px;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```

#### PreProcessed imports
Split your Scss into many files, and import them into one file. Sass will then compile them into a single files so there is only one http request for your `css`.

##### Your less files:

###### variables.less
```less
@nav-height: 48px;
```

###### nav.less
```less
.nav {
    background: #4078c0;
    height: @nav-height;
    padding: 0 6px;
}
```
 
###### link.less
```less
.link {
    font-size: 12px;
    // Get the text vertically centered in the nav.
    line-height: @nav-height;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```

###### index.less
```less
@import 'variables.less';
@import 'nav.less';
@import 'link.less';
```
 
##### Your output:
 
```css
.nav {
    background: #4078c0;
    height: 48px;  
    padding: 0 6px;
}

.link {
    font-size: 12px;
    line-height: 48px;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```
 
#### Mixins
Predefined set of styles that can take arguments that adjust it's output. Maybe your headings have the same font-family and font-weight, but have a variable font-size. You could make a mixin that accepts a size argument and the outputs all three properties.

##### Your less files:

###### variables.less
```less
@nav-height: 48px;
```

###### mixins.less
```less
.link-style(@font-size) {
    font-size: $font@size;
    font-family: sans-serif;
    font-weight: 100;
}
```

###### nav.less
```less
.nav {
    background: #4078c0;
    height: @nav-height;
    padding: 0 6px;
}
```
 
###### link.less
```less
.link {
    .link-style(12px);
    // Get the text vertically centered in the nav.
    line-height: @nav-height;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```

###### index.less
```less
@import 'variables.less';
@import 'mixins.less';
@import 'nav.less';
@import 'link.less';
```
 
##### Your output:
 
```css
.nav {
    background: #4078c0;
    height: 48px;  
    padding: 0 6px;
}

.link {
    font-size: 12px;
    font-family: sans-serif;
    font-weight: 100;
    line-height: 48px;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```
 
#### Mixins as Functions
Less has mixins that act like functions but they can not be called as a value of a property. Any variables that are defined in the mixin are in the same scope as where you call the mixin, so you have access to those variables. Why Less, why you do dis to me.

##### Your less files:

###### variables.less
```less
// Setting @root-font-size to 100% so that the $rem-base is 16px.
$root-font-size : 100%;
$rem-base       : 16;
@nav-height     : 48px;
```

###### functions.less
```less
// Less can not strip unit at all so we are only going to pass in numbers
.rem(@value, @base-value: @rem-base) {
  @rem: @value / @base-value * 1rem;
}

// This is a less version of a function from foundation for apps: https://github.com/zurb/foundation-apps/blob/master/less/helpers/_functions.less
```

###### mixins.less
```less
.link-style(@font-size) {
    .rem(@font-size);
    font-size: @rem;
    font-family: sans-serif;
    font-weight: 100;
}
```

###### elements.less
```less
body {
    font-size: @root-font-size;
}
```

###### nav.less
```less
.nav {
    background: #4078c0;
    height: @nav-height;
    .rem(6);
    padding: 0 @rem;
}
```
 
###### link.less
```less
.link {
    .link-style(12);
    // Get the text vertically centered in the nav.
    line-height: @nav-height;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```

###### index.less
```less
@import 'variables.less';
@import 'functions.less';
@import 'mixins.less';
@import 'elements.less';
@import 'nav.less';
@import 'link.less';
```
 
##### Your output:
 
```css
body {
    font-size: 100%;
}

.nav {
    background: #4078c0;
    height: 3rem;  
    padding: 0 .375rem;
}

.link {
    font-size: .75rem;
    font-family: sans-serif;
    font-weight: 100;
    line-height: 3rem;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```

#### Nesting
Probably the most widely used feature of all preprocessors, and also the most deadly feature. Nesting lets you write a new selector with properties and all inside of another selector. This would then our put a nested selector.

##### Your less files:

###### variables.less
```less
// Setting $root-font-size to 100% so that the $rem-base is 16px.
$root-font-size : 100%;
$rem-base       : 16px;
@nav-height     : 48px;
```

###### functions.less
```less
// Strip Unit
// Removes the unit (e.g. px, em, rem) from a value, returning the number only.
// @param {number} $num - Number to strip unit from.
// @return The same number, sans unit.
@function strip-unit($num) {
  @return $num / ($num * 0 + 1);
}


// Convert to Rem
// Converts a pixel value to matching rem value. *Any* value passed, regardless of unit, is assumed to be a pixel value. By default, the base pixel value used to calculate the rem value is taken from the `$rem-base` variable.
// @param {number} $value - Pixel value to convert.
// @return A number in rems, calculated based on the given value and the base pixel value.
@function rem($value, $base-value: $rem-base) {
  $value: strip-unit($value) / strip-unit($base-value) * 1rem;
  @if ($value == '0rem') { $value: 0; } // Turn 0rem into 0
  @return $value;
}

// Both off these functions come from foundation for apps: https://github.com/zurb/foundation-apps/blob/master/less/helpers/_functions.less
```

###### mixins.less
```less
.link-style(@font-size) {
    font-size: rem(@font-size);
    font-family: sans-serif;
    font-weight: 100;
}
```

###### elements.less
```less
body {
    font-size: $root-font-size;
}
```

###### nav.less
```less
.nav {
    background: #4078c0;
    height: @nav-height;
    padding: 0 rem(6px);
    
    .logo {
        background-image: src('logo.png');
        background-size: contain;
        background-repeat: no-repeat;
        background-position: center;
    
        height: rem(24px);
        width: rem(62px);
    }
}
```
 
###### link.less
```less
.link {
    .link-style(12px);
    // Get the text vertically centered in the nav.
    line-height: @nav-height;
    color: #ccc;
    
    &:hover {
        color: #fff;
    }
}
```

###### index.less
```less
@import 'variables.less';
@import 'functions.less';
@import 'mixins.less';
@import 'elements.less';
@import 'nav.less';
@import 'link.less';
```
 
##### Your output:
 
```css
body {
    font-size: 100%;
}

.nav {
    background: #4078c0;
    height: 3rem;  
    padding: 0 .375rem;
}

.nav .logo {
    background-image: src('logo.png');
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;

    height: 1.5rem;
    width: 3.875rem;
}

.link {
    font-size: .75rem;
    font-family: sans-serif;
    font-weight: 100;
    line-height: 3rem;
    color: #ccc;
}

.link:hover {
    color: #fff;
}
```


### Stylus

### PostCSS



Lets talk about the whole preprocessor (Less, Stylus, Sass/Scss) vs postprocessor (PostCSS) discussion that is happening in the ui development community currently. There is not really a difference between the two and the reality is that they are both postprocessors. If you have some `PostCSS` code that has a few plugins in it
