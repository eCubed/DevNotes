# Site-wide CSS Layout for Angular App

## Preliminaries

Before we dive into styling our site, we must have set up our SCSS infrastructure according to [this document](scss-infrastructure-in-angular.md).

The goal of this article is to set up our entire website's layout. With the nav bar from the previous article, the file  'app.component.html` is as 
follows:

```html
<div>
  <h1>
    Welcome to {{ title }}!
  </h1>
</div>
<app-nav-bar></app-nav-bar>
<router-outlet></router-outlet>
```

The above code is ultimately the exact and only contents of the `<body>` tag in the final html because our `index.html` is this:

```html
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>My App</title>
  <base href="/">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
</head>
<body>
  <app-root></app-root>
</body>
</html>
```

Our entire application will be substituted for `<app-root>` in the final html. It is important to know that the entire markup that will fill 100%
of the page is in `app.component.html`, and that the `<body>` tag's only children are whatever we put in `app.component.html`.

## Vision
We are going to create a very simple layout for this article with only the nav bar and the main content (`<router-outlet>` output) below it. We will
also add a solid background color to the body, and offsett the top of the `<app-root>` by a few pixels, and then center it.

## Styling the body tag

In this example, we will style the body tag to have a particular color because we want to put the main content (white background) over it. We do this
in `styles.scss`. We'll set the font as well:

`styles.scss`
```scss
@import './reset';
@import './variables';

body {
  background-color: $main-color;
  font-family: Tahoma;
}
```

## Modifying app.component.html

We need to modify app.component.html first. We need to add some `<div>` tags to create some infrastructure to target shortly with some styles.

`app.component.html`
```html
<div class="app-container">
  <app-nav-bar></app-nav-bar>
  <div class="router-outlet-container">
    <router-outlet></router-outlet>
  </div>
</div>
```

We now wrap the whole application with a div with class `app-container`. This div is going to be the only child of the body tag.

We still have the navigation bar `<app-nav-bar>` as the top element.

Notice that we now wrap the `<router-outlet>` tag with another div with class `router-outlet-container`. This way, we have a wrapping div that can
provide some padding and other stiles for whatever route's content that would be put in place of `<router-outlet>`.

## Styling app.component.html

We could have put the style of this component right at `app.component.scss`, but we will instead put it in `/src/app/scss/_layout.scss`, and then
import that from `/src/app/scss/styles.scss`. This is debatable, however, we regard the layout as universal styling not specific to any component,
thus we regard site layout as global.

`_layout.scss`
```scss
@import 'variables';

.app-container {
  position: relative;
  width: 80%;
  margin: auto;
  background-color: white;
  top: #{3 * $general-padding};

  & > * {
    width: 100%;
  }

  & > .router-outlet-container {
    padding: $general-padding;
    min-height: 400px;
  }
}
```

There are a few important things to consider:

* The whole container `.app-container` must be positioned relative so that we can set its top a few pixels below the top of the browser pane.
* Width being 80% and margin set to auto makes the app skinnier than the browser window and align center automatically.
* We set the background to white because the body background would be something else, and we didn't want to inherit that background.
* The width of everything that is an immediate child of `.app-container` should fill `.app-container`, thus 100%.

We will then need to modify `styles.scss` and import `layout` into it. 

`styles.scss`
```scss
@import './reset';
@import './layout';

body {
  background-color: $main-color;
  font-family: Tahoma;
}
```

## Styling the Nav Bar

Right now, our navigation bar is practically an unordered list that will display vertically. We want a right-aligned navigation bar that will display
menu items horizontally.

`nav-bar.component.html`
```html
<nav>
  <ul>
    <li>
      <a routerLink="/">Home</a>
    </li>
    <li>
      <a routerLink="/about">About</a>
    </li>
    <li>
      <a routerLink="/contact">Contact</a>
    </li>
  </ul>
</nav>
```

`navbar-component.scss`
```scss
@import '~src/scss/variables';
@import '~src/scss/utils/clearfix';

nav {
  ul {
    width: 100%;
    background-color: #0022b3;

    @include clearfix();

    li {
      float: right;
      list-style: none;
      padding: $general-padding;

      a {
        display: block;
        text-decoration: none;
        font-weight: bold;
        color: white;
      }
    }
  }
}
```

* Set the width of the unordered list to 100% so we can later horizontally align the list items.
* Set it to a background color darker than the body's background color.
* Include the clearfix rule because we'll need to float the list items right. Browsers, for some reason, will think that the parent of floated elements
would have a height of 0 or some small number of pixes, and those floated list items would be covered by subsequent by subsequent html.
* On the list item, list style none removes the bullet, and we will need to give it some padding.
* The link inside the list item MUST be set to `display: block` so that it would fill its parent vertically. We set the color of the link text to white
because the background is dark.






