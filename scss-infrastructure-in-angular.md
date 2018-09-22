# SCSS Infrastructure in an Angular 6.x App

When we first created our Angular app and specified that we want SCSS, not CSS, we were also given file `/src/styles.scss` where we can put rules that would be
accessible by any component. But putting everything in this file is not a very organized practice. We will need separate files to store global values like colors
and general padding. We will need other files that would declare styling for html elements that would automatically apply in general.

How does Angular know about this `/src/styles.scss` file and all of its rules are automatically accessible and thus, will apply to all components that use those
rules? It is referenced from `angular.json` in various places. This is where CSS is "manufactured" into the final single CSS file that the browser reads. I this
file, we import our global styles, reset CSS, and variables. But, we can change this file - name and location that makes more sense for us. We can create an scss
folder underneath `/src` and move styles.scss to it. But now, Angular is still looking for `/src/styles.scss`, so we'll need to go to `angular.json` and
edit all the references to point to `/src/scss/style.scss` instead. We would need to recompile the application, though.

# _variables.scss
We need to create an scss file that stores JUST variables and their values to later be used from components. The file will be at `/src/scss/_variables.scss`:

```css
$main-color: #0000cc;
$danger-color: #cc0000;
$success-color: #00cc00;
$warning-color: #cccc00;

$general-padding: 10px;
```

We do NOT need to import `variables` into `styles.scss`. We will need to explicitly import it to every component that needs them!

As a note, SCSS files that have a leading underscore are meant to be imported into other SCSS files. NEVER will we import an SCSS file that doesn't start with an
underscore. However, when we reference such a file, we do not need to include the underscore or the `.scss` extension!

# _reset.scss
Resets are very controversial no matter which one we select and apply to our app. However, we will need to have a way to automatically remove paddings and margins
so that we can later explicitly set them. For this article, as supposedly wrong as it may seem, we'll choose the most minimal reset of all:

```css
* {
   padding: 0;
   margin: 0;
   box-sizing: border-box;
}
```

The rule `box-sizing: border-box;` means that border and padding will be included when we set the element's width and height. Many find that this approach
makes more sense and more predictable when laying out rectangular areas against each other to fit snug in their parents' width.

We will need to import `resets` into `styles.scss` for it to take effect before any component SCSS kicks in.

## Utilities
Mixins and functions typically go in the folder `src/scss/utilities/`. Two utilities that we'll need quite often are clearfix and tintshade:

`_tintshade.scss`
```css
@function tint($color, $percentage) { 
  @return mix(white, $color, $percentage); 
} 

@function shade($color, $percentage) { 
  @return mix(black, $color, $percentage); 
}
```

`_clearfix.scss`
```scss
@mixin clearfix() { 
  &:before, 
  &:after { 
      content: ""; 
      display: table; 
  } 
  &:after { 
      clear: both; 
  } 
}
```

These utilities do NOT need to be imported into the `styles.scss` file because we will NOT be creating CSS rules there. We will need to nit-pick-import these
utilities in every component that needs them!

## Importing From Components

We used to need to reference base SCSS in components this way: `@import '../../../../../scss/variables;'` which was an eyesore and was always a second-guessing
effort that was more often wrong than right. Assuming that the `/scss` folder was actually at `/src/scss` which it should be, we could now reference this way:

`@import '~src/scss/variables';`

... no matter how deep in the project's folder structure that component's SCSS is!

## What Exactly is Global Styling?
This is another controversial topic in Angular CSS, and we'll get to why in a bit. Global styles are intended to declare CSS rules on specific elements or selectors
that would apply all across the board, all the way down to the childmost component in the entire app. Depending on the size and the needs of an app, the number of
global SCSS files will vary, however, they should be named to reflect their contents. For example, there could be a global SCSS file called `_links.scss` which
will style the `<a>` element and its pseudo selectors like `:hover`. No other element's styling would be included in this file. Now, doing this can be a bit
controversial at least two ways. If we have components that want their own look for their own `<a>` tags, then we'd always have to override the styling for the
`<a>` tag in that component's SCSS file with `!important`. For us, the developers, this can be tedious because we'd have to know all of the CSS rules stated in
`_links.scss` to know what to override. Another abuse of a global SCSS file is, even for our links example, is to put every possible "selector targeting" that an
`<a>` tag would ever be in across the entire app. For example, nit-pick when an `<a>` tag is an immediate child of an `<li>` element, and style it one way, but
style it another way if it's a child of a `<p>` tag whose class is "emphasis". This latter violation actually makes the global style need to be aware of the entire
application, but it's supposed to be "global", meaning "should apply everywhere just like that."

We will find situations where we want to apply global styles, for example, it is common for all input elements to look the same all across the board, and we better
not require a component to drastically alter its look. Again, it's a global CSS file that shouldn't care about where in the HTML, even relatively, the elements are
that they're styling.

Global styles will all need to be imported to the `styles.scss` ONLY, NEVER in any component. Importing these global styles in multiple components WILL cause
duplicate CSS rules, and it would confuse the browser and ultimately, not render the page and its contents correctly.

## Next Steps

We only covered how to set up the file-folder structure of SCSS files, knowing what and what not to import from various SCSS files, and how to import correctly.
For global styles, we finalized that we will only style by tag name or class name, and NEVER target anything depending on what its parents and other ascendants
are (because that forces something global to be dependent on whatever app it will be styling).

We haven't covered the following:

* The `:host` pseudo selector
* View scope
* Vendor-specific SCSS
* Deciding where to put css rules - component scss, styles.scss, or some global scss file

