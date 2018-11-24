# Flex Layout - Responsive Page

We will work off the simple page layout we've started in the previous journal and make it responsive. The layout created so far has Flex Layout declarations
with only a wide and the upper range of medium viewport widths. If we shrink the width of the browser to under 400px which is considered a small width,
our layout will be too squashed, especially the sidebar and main content. Everything declared to be layed out in rows will still be displayed in rows
in a small width viewport, whereas a vertical alignment for most of the content would make more sense.

There are a few things we'll need to do and consider before declaring responsiveness in our markup:

* Part of responsiveness is deciding which elements to *show up at all* or conversely *hide* depending on the viewport width. For example, when the page
is viewed on a desktop, we might want to show all the links and the logo. However, we might want hide them when viewing on a phone in "portrait mode"
but instead show a menu icon that when clicked, would show the menu.
* As mentioned, many of the children layed out in rows will need to be layed out in columns on a narrower viewport.
* There are content that may call for fewer columns per row for smaller width. For example, an image gallery may have 6 images per row for a large
viewport, 4 for a medium, and possibly 2 for small.
* Let's add a few more pieces of content to our markup to see responsiveness in action. We'll add links to the sidebar and the links menu in the header.
Let's also add some content in the main content section that warrants responsiveness.

We will start from where we left off in the previous journal where we explored nested containers with the starting page layout.

```html
<div class="page" fxLayout="column">
  <div class="header" fxLayout="row" fxLayoutAlign="space-between">
    <div class="logo">
      Logo
    </div>
    <div class="links">
      Links
    </div>
  </div>
  <div class="content" fxLayout="row">
    <div class="sidebar" fxFlex="1 0 10px">
      Sidebar
    </div>
    <div class="main" fxFlex="4 0 10px">
      Main content
      <p>
        This is a paragraph to make this section taller.
      </p>
      <p>
        Here's another paragraph to make it even taller still.
      </p>
    </div>
  </div>
  <div class="footer">
    Footer
  </div>
</div>
```

## Flex Layout Breakpoints

Flex Layout comes with media queries internally so we won't have to crowd our CSS with them. Just like how we declaratively specify relative widths and
layouts in markup, we *also* declare breakpoints in markup, and they're *surprisingly* very easy. First, we need to know the aliases for the ranges of
pixel widths.

* **xs**, 0 - 599px
* **sm**, 600 - 959px
* **md**, 960 - 1279px
* **lg**, 1280 - 1919px
* **xl**, 1920 - 5000px

Those are the threshhold values *by default*. There is a way to customize the ranges, but that's a more advanced topic we'll explore in a future journal.

We need to know these names because these are *exactly* what we'll be putting in our markup.

## Applying Flex Layout Breakpoints

Let's focus on the main content where we have the sidebar and the main content area side-by-side because we set `fxLayout="row"`. When we're
in the small (sm) viewport, we want to display them vertically.

```
<div class="content" fxLayout="row" fxLayout.sm="column">
  <div class="sidebar" fxFlex="1 0 10px">
    ...
  </div>
  <div class="main" fxFlex="4 0 10px">
    ...
  </div>
</div>
```
We have added `fxLayout.sm="column"` to the content div. Specifying a breakpoint is a matter off appending `.{breakpoint}` to a Flex Layout directive.
Then, we can specify the values that pertain *only* to that breakpoint.

If we run our application now when the browser is wide, we'll see the layout as usual. If we shrink it a little bit, when the width falls within
600 - 959px (sm), we will see that the sidebar and main content are now stacked vertically. However, there is a problem - the heights of bothsidebar and
main divs are now sqashed to 10px. This is because we're now in the column layout, and we did not set a height for the content div. We'll leave
it this way for now, but there is another problem.

If we shrink the browser further until it reaches less than 600px wide, we'll see that the side bar and the main divs are side-by-side again. Note that
when we said `fxLayout.sm="column"`, this means any with between 600 - 959px will get the column layout. Anything outside that range will not apply the
column layout. Sure, we could have added another `fxLayout` declaration, `fxLayout.xs="column"` and it would work - xs widths will now apply a column
layout, but there is a better way.

## Comparison Flex Layout Breakpoints

Angular Flex Layout also comes with convenient breakpoints that would apply specific Flex Layout declarations for everything that is greater than or less
than a specific width. In our case when we want to apply a column layout to small and extra-small, this means widths between 0 - 959px. In order to target
this breakpoint in one shot, we can declare `fxLayout.lt-md="column"`. `lt-md` is "less than medium". Medium  begins at 960px, so anything less than
medium really covers extra-small and small.

* **lt-sm** - 0 - 599px. Anything less than small, which really is equivalent to .xs.
* **lt-md** - 0 - 959px. Anything less than medium, which includes .xs and .sm.
* **lt-lg** - 0 - 1279px. Anything less than large, includes .xs, .sm, and .md.
* **lt-xl** - 0 - 1919px. Anything less than extra-large, includes .xs, .sm, .md, and .lg.

There are also "greater-than" comparison breakpoints should we find them more convenient depending on how we perceive breakpoints.

* **gt-xs** - 600px +, Anything .sm and larger.
* **gt-sm** - 960px +, Anything .md and larger.
* **gt-md** - 1280px +, Anything .lg and larger.
* **gt-lg** - 1920px +, Anything .xl and larger (if anything).

So, let's finally fix our content layout:

```html
<div class="content" fxLayout="row" fxLayout.lt-md="column">
  <div class="sidebar" fxFlex.gt-sm="1 0 10px">
     ...
  </div>
  <div class="main" fxFlex.gt-sm="4 0 10px">
    ...
  </div>
</div>
```

We want column layout for widths less than medium, so that's .xs and .sm, hence we specify `fxLayout.lt-md="column"`. We want the side-by-side layout to
apply for anything greater than small, which is medium and larger, thus, we specify the relative widths with `fxFlex.gt-sm`. If we now run our app,
we get the desired effect.

## Hiding and Showing Elements by Breakpoint

Now, we'll address showing and hiding elements depending on their width. The most popular use case for this is in the application's toolbar. Typically,
we would hide the navigation on narrow widths, but show an "expand menu" icon instead.

Let's first create the menu items in the links div:

```html
<div class="links" fxLayout="row" fxLayoutGap="10px">
  <a href="#">Home</a>
  <a href="#">Contact</a>
  <a href="#">Services</a>
</div>
```

We'll also style those anchor tags:

```css
.links {
  box-sizing: border-box;
  padding: 10px;
  background-color: #ffd149;

  & > a {
    font-weight: bold;
    text-decoration: none;    
  }
}
```
We want both the logo and links div to disappear when the width is less than medium. In order to do that, we'll use the `fxHide` directive on those divs.

```html
<div class="header" fxLayout="row" fxLayoutAlign="space-between">
    <div class="logo" fxHide.lt-md>
      ...
    </div>
    <div class="links" fxHide.lt-md fxLayout="row" fxLayoutGap="10px">
      ...
    </div>
  </div>
```

We use `fxHide.lt-md` to indicate that the container should be hidden *when* the width is less than medium. If we resize our browser small enough, we
will see that the logo and links divs disappeared. Let's NOT confuse `fxHide` with `*ngIf="false"` because they don't actually mean the same thing.
With `*ngIf="false"`, it means that the element is *fully absent* from the DOM. With, `fxHide` the element that applies it *is still in* the DOM.
It just gets a styling of `display: none`.

Now, typically, when the viewport width is narrow and the logo and nav items disappear, a menu icon is supposed to show up instead. So, we'll add that
div to the header div. We want that menu icon to hide when the width is greater than small, so it wouldn't show up if the width were medium or greater.
We also want that menu item to align to the right.

```html
<div class="header" fxLayout="row" fxLayoutAlign="space-between" fxLayoutAlign.lt-md="end">
  <div class="logo" fxHide.lt-md>
    Logo
  </div>
  <div class="links" fxHide.lt-md fxLayout="row" fxLayoutGap="10px">
    <a href="#">Home</a>
    <a href="#">Contact</a>
    <a href="#">Services</a>
  </div>
  <div class="menu-icon" fxHide.gt-sm>
    Menu
  </div>
</div>
```
We now add the menu-icon div, and `fxHide.gt-sm` will hide it for medium and wider widths. In order to align the menu-icon div to the right, *only when*
the viewport width is less than medium, we add another `fxLayoutAlign` declaration to the parent header - `fxLayoutAlign.lt-md="end"`, which will only
apply when the width is small or less, when we have only the menu-item showing.

## Managing Toggle on Click
Now let's focus on the menu item. Yes, we get it to show up when the width is narrow, and it hides when the width is wider. But we should be able to click
it and a menu should pop up. It should also be a toggle so that clicking it over and over again should show and hide the menu.





