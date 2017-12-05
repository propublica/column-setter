# ColumnSetter

ColumnSetter is a Sass tool that lets you easily set up a custom column structure for your website and build a layout that aligns to it. It uses just a few simple functions and mixins to generate percentage widths based on your settings.

## Setup

Begin by saving `_columnsettings.scss` and `_columnsetter.scss` in the same directory as your main Sass file.

In `_columnsettings.scss`, customize your column structure’s proportions by editing the values of the four variables at the top of the file. Use only numbers for values, no units (e.g. “10”, not “10px”). **Don’t delete any of these variables!** For any you don’t need to use, just assign a value of 0. Some sample values:

```
$mar: 68; // Margin width
$col: 68; // Column width
$gut: 26; // Gutter width
$pad: 13; // Padding width
```

Next, customize the layout’s breakpoints by editing the `$breakpoints` map. You can define as many (or as few) breakpoints as you like, and name them whatever you want. Just be sure to:

* use the syntax shown below,
* keep the breakpoints in order (smallest to largest), and
* include a name (e.g. “xl”), column count (“cols”, unitless) and minimum measurement (“min”, with units) for each breakpoint.

All breakpoints use the same proportions specified in the variables above, but margin widths can be optionally customized for each breakpoint. Here’s a sample `$breakpoints` map with five breakpoints:

```
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

```
@import "_columnsettings.scss";
@import "_columnsetter.scss";
```

**To get the most out of ColumnSetter, using it in conjunction with `* { box-sizing: border-box; }` is strongly recommended.**

## Usage

TK

### `grid()`

`grid()` is a mixin used to establish the context for your column structure. The first (and only required) argument it takes is the class name of the layout’s container. If, for example, your layout is wrapped in a `div` with a class of `main-content`, you’ll invoke `grid()` like so:

```
@include grid( main-content );
```

This establishes the container’s horizontal margins for each of the breakpoints you specified in `_columnsettings.scss`, and it compiles to something like this:

```
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

```
@include grid( main-content, overlay );
```

### `colspan()`

`colspan()` is a function used to generate percentage widths in proportion with the column structure. To use it on an element, you’ll need to know how many columns wide the element’s container is. For example, to specify a width of 6 columns for an element inside a container that’s 12 columns wide:

```
.example {
  width: colspan( 6, 12 ); // Make this 6 columns wide out of 12
}
```

Which compiles to something like this:

```
.example {
  width: 48.82033%;
}
```

`colspan()` will also take the arguments `p` and `g`, for padding width and gutter width:

```
img.inset {
  float: left;
  width: colspan( 2, 8 );
  padding: colspan( p, 8 );
  margin-right: colspan( g, 8 );
  margin-bottom: colspan( g, 8 );
}
```

And for more granular control, you can combine `colspan()` functions:

```
.example {
  margin-left: colspan( 6, 12 ) + colspan( g, 12 );
}
```

### `breakpoint-min()`, `breakpoint-max()`, `breakpoint-range()`

TK

### `full-width`

`full-width` is a mixin used to break an element out of the layout and take up the full width of the viewport. It doesn’t take any arguments; its output is always the same. Use it like so:

```
.example {
  @include full-width;
}
```

Which compiles to:

```
.example {
  width: auto;
  margin-left: calc(-50vw + 50%);
  margin-right: calc(-50vw + 50%);
}
```

## Tips and tricks

TK

* You’re not confined to the width of the container! (e.g. `colspan( 12, 6 )`)