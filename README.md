# Column Setter

Column Setter is a [Sass](https://github.com/sass/sass) tool that lets you easily set up a custom responsive grid for your website and build a float- or flexbox-based layout that aligns to it. It uses one simple function and a small handful of optional mixins to generate CSS widths based on your settings. And for the most part, it leaves the structure of your HTML and CSS entirely up to you.

## v2.0 Updates

* Unit values are now supported in the grid property variables (`$mar`, `$col`, `$gut`, `$pad`) in  `_column-settings.scss`.
* All grid properties can now be customized per breakpoint in the `$breakpoints` map in `_column-settings.scss`.
* `colspan()` can now natively generate negative width values (`-`) and add a gutter to width values (`+g`).
* A wider range of errors now can be detected, including missing or incorrectly formatted settings.

## Table of Contents

* **[Setup](#setup)**
* **[Implementation](#implementation)**
* **[Usage](#usage)**
    * [`colspan()`](#colspan)
    * [`grid()`](#grid)
    * [`breakpoint-min()`](#breakpoint-min), [`breakpoint-max()`](#breakpoint-max), [`breakpoint-range()`](#breakpoint-range)
    * [`full-width`](#full-width)
* **[Tips and tricks](#tips-and-tricks)**

## Setup

Begin by saving `_column-settings.scss` and `_column-setter.scss` in the same directory as your main Sass file.

### Grid proportions

In `_column-settings.scss`, customize your grid’s proportions by editing the values of the four variables at the top of the file (`$mar`, `$col`, `$gut`, `$pad`). These establish the spatial relationship between the grid’s properties. For an entirely fluid grid, use all unitless numbers (e.g. “`$mar: 4; $col: 3;`”). These will cause [`colspan()`](#colspan), Column Setter’s central function, to generate `%` values. Otherwise, units are allowed (e.g. “`$gut: 1em;`”), as is a mix of unitless and units (e.g. “`$col: 3; $gut: 1em;`”). Mixed units will cause `colspan()` to generate `calc()` values. When using units, keep these caveats in mind:

* Giving `$col` a unit value is allowed but not recommended.
* If `$mar` and `$gut` both have unit values, giving `$col` any unitless value will make columns fill the available space, as if they were `1fr` in CSS Grid. Likewise, if `$mar` and `$col` both have unit values, giving `$gut` any unitless value will make gutters fill the available space.
* Making `$mar` or `$pad` unitless will have no effect unless `$col` and/or `$gut` are also unitless.

However you decide to define their values, **don’t delete any of these variables.** For any you don’t need to use, just assign a value of 0. The Column Setter demo (`demo.html`) creates a completely fluid grid by using all unitless values:

```scss
$mar: 4; // Margin width
$col: 4; // Column width
$gut: 2; // Gutter width
$pad: 1; // Padding width
```

### Breakpoints

Once you’ve chosen values for the grid proportions, customize the layout’s breakpoints by editing the `$breakpoints` map. You can define as many (or as few) breakpoints as you like, and name them whatever you want. Just be sure to:

* use the syntax shown below
* keep the breakpoints in order (smallest to largest)
* include a name (e.g. `xl`), number of columns (`cols`, unitless) and minimum width (`min-width`, with units, such as `em` or `px`) for each breakpoint

Any or all of the grid’s proportions can optionally be customized per breakpoint, using the keys `margin`, `column`, `gutter`, and `padding`. Breakpoints without those customizations will use the same proportions specified in the variables above. Here’s a sample `$breakpoints` map with five breakpoints:

```scss
$breakpoints: (
  xs: ( cols:  4, min-width:  0,   padding: 2em ), // Includes optional custom padding
  sm: ( cols:  6, min-width: 30em, margin: 5 ),    // Includes optional custom margin
  md: ( cols:  8, min-width: 40em ),
  lg: ( cols: 12, min-width: 50em ),
  xl: ( cols: 16, min-width: 60em )
);
```

## Implementation

Once your settings are in place, import `_column-settings.scss` and `_column-setter.scss` (in that order) into your main Sass file:

```scss
@import "_column-settings.scss";
@import "_column-setter.scss";
```

## Usage

**To get the most out of Column Setter, using it in conjunction with `* { box-sizing: border-box; }` is strongly recommended.** Column Setter is lean and mean, packing a lot of power into just one function and five optional mixins.

### `colspan()`

`colspan()` is a function used to generate width values that align with the grid. To use it on an element, you’ll need to know how many columns wide the element’s container is. For example, to specify a width of six columns for an element inside a container that’s 12 columns wide:

```scss
.example {
  width: colspan( 6, 12 ); // 6 columns wide out of 12
}
```

Depending on your grid settings, the above code might compile to something like this:

```css
.example {
  width: 48.82033%;
}
```

Or something like this:

```css
.example {
  width: calc((((100% - 11em) / 12) * 6) + 5em);
}
```

`colspan()` will also take the arguments `p` (padding) and `g` (gutter):

```scss
img.inset {
  float: left;
  width: colspan( 2, 8 );
  padding: colspan( p, 8 );
  margin-right: colspan( g, 8 );
  margin-bottom: colspan( g, 8 );
}
```

A gutter width can optionally be added by including `+g`:

```scss
.example {
  margin-left: colspan( 6+g, 12 ); // 6 columns wide out of 12, plus an extra gutter
}
```

Widths can also be made negative (useful for negative margins) by including `-`:

```scss
.example {
  margin-left: colspan( -6+g, 12 );
}
```


### `grid()`

`grid()` is a mixin used to establish the context for your grid. It sets the vertical margins for your layout at all breakpoints. The first (and only required) argument it takes is the class name of the layout’s container. If, for example, your layout is wrapped in a `div` with a class of `main-content`, you’ll invoke `grid()` like so:

```scss
@include grid( main-content );
```

This establishes the container’s horizontal margins for each of the breakpoints you specified in `_column-settings.scss`, and it compiles to something like this:

```css
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

You can also optionally include `overlay` as a second argument, which will put a translucent overlay of the grid on top of your layout so you can make sure everything is lining up correctly:

```scss
@include grid( main-content, overlay );
```

*Please note that the overlay will only show padding widths if the grid is completely fluid (i.e. the initial variables in the settings are all unitless).*

### `breakpoint-min()`, `breakpoint-max()`, `breakpoint-range()`

The three `breakpoint` mixins generate media queries based on the `$breakpoints` settings specified in `_column-settings.scss`.

```scss
@include breakpoint-min( xs ) { ... }
@include breakpoint-max( sm ) { ... }
@include breakpoint-range( md, lg ) { ... }
```

The above code compiles to something like this:

```css
@media screen and (min-width: 0) { ... }
@media screen and (max-width: 30em) { ... }
@media screen and (min-width: 40em) and (max-width: 50em) { ... }
```

### `full-width`

`full-width` is a mixin used to break an element out of the layout and take up the full width of the viewport. It doesn’t take any arguments; its output is always the same. Use it like so:

```scss
.example {
  @include full-width;
}
```

The above code compiles to:

```css
.example {
  width: auto;
  margin-left: calc(-50vw + 50%);
  margin-right: calc(-50vw + 50%);
}
```

*Please note that `full-width()` only works on grids that are centered in the viewport.*

## Tips and tricks

* For the most part, Column Setter code will compile even if you make a mistake—the invalid code will simply be ignored. If something isn’t working the way you expected, look for a `WARNING` on the command line. Column Setter can recognize common mistakes and help you troubleshoot them.

    ```sh
    WARNING: 'breakpoint-min(Gerald)' is invalid because 'Gerald' is not a valid breakpoint name. A 'breakpoint-min' mixin call must contain a valid breakpoint name, e.g. 'breakpoint-min(small)'. Current valid breakpoint names: xs, sm, md, lg, xl.
    ```

* Elements are not confined to the width of their containers. Want an eight-column element inside a six-column container? No problem:

    ```scss
    width: colspan( 8, 6 );
    ```

   Want to center it? Remember, `colspan()` values can be made negative:

    ```scss
    margin-left: colspan( -1+g, 8 );
    ```

* Each breakpoint is required to specify a number of columns, but those column counts don’t all have to be unique. You might want certain elements to change at a certain breakpoint without changing the rest of the layout. Just add another breakpoint in `_column-settings.scss` with the same number of columns as the one before or after it.

    ```scss
    $breakpoints: (
      small:  ( cols:  6, min-width:  0 ),
      medium: ( cols: 10, min-width: 30em ),
      large:  ( cols: 10, min-width: 35em )
    );
    ```