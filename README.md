# ColumnSetter

ColumnSetter is a [Sass](https://github.com/sass/sass) tool that lets you easily set up a custom responsive column structure for your website and build a float-based layout that aligns to it. It uses just a few simple functions and mixins to generate CSS percentage widths based on your settings, and it leaves the structure of your HTML entirely up to you.

## Contents

* **[Setup](#setup)**
* **[Implementation](#implementation)**
* **[Usage](#usage)**
    * [`colspan()`](#colspan)
    * [`grid()`](#grid)
    * [`breakpoint-min()`](#breakpoint-min), [`breakpoint-max()`](#breakpoint-max), [`breakpoint-range()`](#breakpoint-range)
    * [`full-width`](#full-width)
* **[Tips and tricks](#tips-and-tricks)**

## Setup

Begin by saving `_columnsettings.scss` and `_columnsetter.scss` in the same directory as your main Sass file.

In `_columnsettings.scss`, customize your column structure’s proportions by editing the values of the four variables at the top of the file. Use only numbers for values, no units (e.g. `10`, not `10px`). **Don’t delete any of these variables!** For any you don’t need to use, just assign a value of 0. Some sample values:

``` sh
$mar: 68; // Margin width
$col: 68; // Column width
$gut: 26; // Gutter width
$pad: 13; // Padding width
```

Next, customize the layout’s breakpoints by editing the `$breakpoints` map. You can define as many (or as few) breakpoints as you like, and name them whatever you want. Just be sure to:

* use the syntax shown below,
* keep the breakpoints in order (smallest to largest)
* include a name (e.g. `xl`), column count (`cols`, unitless) and minimum width (`min-width`, with units) for each breakpoint.

All breakpoints use the same proportions specified in the variables above, but margin widths can be optionally customized for each breakpoint. Here’s a sample `$breakpoints` map with five breakpoints:

``` sh
$breakpoints: (
  xs: ( cols:  4, min:  0,   margin: $pad ), // Includes optional custom margin
  sm: ( cols:  6, min: 30em, margin: $gut ), // Another optional custom margin
  md: ( cols:  8, min: 40em ),
  lg: ( cols: 12, min: 50em ),
  xl: ( cols: 16, min: 60em )
);
```

## Implementation

Once your settings are in place, import `_columnsettings.scss` and `_columnsetter.scss` (in that order) into your main Sass file:

``` sh
@import "_columnsettings.scss";
@import "_columnsetter.scss";
```

## Usage

**To get the most out of ColumnSetter, using it in conjunction with `* { box-sizing: border-box; }` is strongly recommended.** ColumnSetter is lean and mean, packing a lot of power into just one function and five optional mixins.

### `colspan()`

`colspan()` is a function used to generate percentage widths in proportion with the column structure. To use it on an element, you’ll need to know how many columns wide the element’s container is. For example, to specify a width of 6 columns for an element inside a container that’s 12 columns wide:

``` sh
.example {
  width: colspan( 6, 12 ); // 6 columns wide out of 12
}
```

Which compiles to something like this:

``` sh
.example {
  width: 48.82033%;
}
```

`colspan()` will also take the arguments `p` (padding) and `g` (gutter):

``` sh
img.inset {
  float: left;
  width: colspan( 2, 8 );
  padding: colspan( p, 8 );
  margin-right: colspan( g, 8 );
  margin-bottom: colspan( g, 8 );
}
```

And for more granular control, you can combine `colspan()` functions:

``` sh
.example {
  margin-left: colspan( 6, 12 ) + colspan( g, 12 );
}
```

### `grid()`

`grid()` is a mixin used to establish the context for your column structure. It sets the vertical margins for your layout at all breakpoints. The first (and only required) argument it takes is the class name of the layout’s container. If, for example, your layout is wrapped in a `div` with a class of `main-content`, you’ll invoke `grid()` like so:

``` sh
@include grid( main-content );
```

This establishes the container’s horizontal margins for each of the breakpoints you specified in `_columnsettings.scss`, and it compiles to something like this:

``` sh
@media screen and (min-width: 0) {
  .content {
    margin: 0 3.38638%;
  }
}
@media screen and (min-width: 30em) {
  .content {
    margin: 0 4.36194%;
  }
}
```

You can also optionally include `overlay` as a second argument, which will put a translucent overlay of the column structure on top of your layout so you can make sure everything is lining up correctly:

``` sh
@include grid( main-content, overlay );
```

### `breakpoint-min()`, `breakpoint-max()`, `breakpoint-range()`

The three `breakpoint` mixins generate media queries based on the `$breakpoints` settings specified in `_columnsettings.scss`.

``` sh
@include breakpoint-min( xs ) { ... }
@include breakpoint-max( sm ) { ... }
@include breakpoint-range( md, lg ) { ... }
```

The above compiles to something like this:

``` sh
@media screen and (min-width: 0) { ... }
@media screen and (max-width: 30em) { ... }
@media screen and (min-width: 40em) and (max-width: 50em) { ... }
```

### `full-width`

`full-width` is a mixin used to break an element out of the layout and take up the full width of the viewport. It doesn’t take any arguments; its output is always the same. Use it like so:

``` sh
.example {
  @include full-width;
}
```

Which compiles to:

``` sh
.example {
  width: auto;
  margin-left: calc(-50vw + 50%);
  margin-right: calc(-50vw + 50%);
}
```

## Tips and tricks

* For the most part, ColumnSetter code will compile even if you make a mistake; the invalid code will simply be ignored. If something isn’t working the way you expected, look for a `WARNING` on the command line. ColumnSetter can recognize common mistakes and help you troubleshoot them.
* Elements are not confined to the width of their containers. Want an 8-column element inside a 6-column container? No problem:

    ``` sh
    width: colspan( 8, 6 )
    ```

   Want to center it? `colspan()` values can be made negative:

    ``` sh
    margin-left: -( colspan( 1, 8 ) + colspan( g, 8 ) );
    ```

* Each breakpoint is required to specify a column count, but those column counts don’t all have to be unique. You might want certain elements to change at a certain breakpoint without changing the rest of the layout. Just add another breakpoint in `_columnsettings.scss` with the same number of columns as the one before or after it.