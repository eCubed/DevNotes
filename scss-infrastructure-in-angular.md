# SCSS Infrastructure in an Angular 6.x App

When we first created our Angular app and specified that we want SCSS, not CSS, we were also given file `/src/styles.scss` where we can put rules that would be
accessible by any component. But putting everything in this file is not a very organized practice. We will need separate files to store global values like colors
and general padding. We will need other files that would declare styling for html elements that would automatically apply in general.

How does Angular know about this `/src/styles.scss` file and all of its rules are automatically accessible and thus, will apply to all components that use those
rules? It is referenced from `angular.json` in various places. This is where CSS is "manufactured" into the final single CSS file that the browser reads. I this
file, we import our global styles, reset CSS, and variables. But, we can change this file - name and location that makes more sense for us. We can create an scss
folder underneath `/src` and move styles.scss to it. But we now, Angular is still looking for `/src/styles.scss`, so we'll need to go to `angular.json` and
edit all the references to point to `/src/scss/style.scss` instead. We would need to recompile the application, though.

# _variables.scss
We need to create an scss file that stores JUST variables and their values to later be used from components. The file will be at `/scss/_variables.scss`:

```css
$main-color: #0000cc;
$danger-color: #cc0000;
$success-color: #00cc00;
$warning-color: #cccc00;

$general-padding: 10px;
```
The app may have different variables and values as needed. Now, this will be referenced *as the first import* in `/scss/styles.scss`:

```scss
@import 'variables';
```

As a note, SCSS files that have a leading underscore are meant to be imported into other SCSS files. NEVER will we import an SCSS file that doesn't start with an
underscore. However, when we reference such a file, we do not need to include the underscore or the `.scss` extension!

