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