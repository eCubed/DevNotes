# Flex Layout - Nested Containers

So far, we have declared a single parent and its children. We didn't treat each child as a parent itself with its own children. In this article, we will explore
how we can nest multiple levels of parent and children to create practically any layout we want. Before we go on, though, let us remember that children of a parent
container may only be layed out in 1 row or 1 column - there is no grid!

We will lay out a basic web page - one with a header, content with sidebar and main content, and a footer. The header also has a logo and a links section. So far,
our markup looks like this:

```html
<div class="page">
  <div class="header">
    <div class="logo">
      Logo
    </div>
    <div class="links">
      Links
    </div>
  </div>
  <div class="content">
    <div class="sidebar">
      Sidebar
    </div>
    <div class="main">
      Main content
    </div>
  </div>
  <div class="footer">
    Footer
  </div>  
</div>
```

We provide some trivial styling so that we can visually see the layout.

```css
.header {
  box-sizing: border-box;
  border: solid 1px #9c7500;
}

.logo {
  box-sizing: border-box;
  padding: 10px;
  background-color: #fed867;
}

.links {
  box-sizing: border-box;
  padding: 10px;
  background-color: #ffd149;
}

.content {
  box-sizing: border-box;
  border: solid 1px #00015b;
}

.sidebar {
  box-sizing: border-box;
  padding: 10px;
  background-color: #8182ff;
}

.main {
  box-sizing: border-box;
  padding: 10px;
  background-color: #e0e0e0;
}

.footer {
  box-sizing: border-box;
  border: solid 1px #007c09;
  padding: 10px;
  background-color: #e8ffaf;
}
```

We can see that there is more than 1 level. For example, header is both a child (of page) and a parent (of logo and links). If we run this example now, we will see
everything stacked vertically. Let's fix a few things. We want the top-level parent's layout to be column. We want the header and the content children to be parents
whose children will be layed out in rows.

```html
<div class="page" fxLayout="column">
  <div class="header" fxLayout="row">
    <div class="logo">
      Logo
    </div>
    <div class="links">
      Links
    </div>
  </div>
  <div class="content" fxLayout="row">
    <div class="sidebar">
      Sidebar
    </div>
    <div class="main">
      Main content
    </div>
  </div>
  <div class="footer">
    Footer
  </div>
</div>
```

The `fxLayout` declarations fixed the layout as we described, but the children of header and content aren't distributed. For the header, we want the logo to left-align
and the links to right-align. Since header has only two children, we can set `fxLayoutAlign="space-between"` on it to get the effect we want. For the content,
we want the sidebar to be 20% width and the main content to be 80%. Since we want to ensure that, we'll initially give both sidebar and main content a small starting
basis of 10px. We'll give sidebar a `flex-grow` of 1, which is the 20% width if the main content's `flex-grow` will be 4. This way, 20% of the leftover space will
be added to the sidebar, and 80% of the leftover space will be given to the main content. Alternatively, we could have set the flex bases to those percentages, set
the `flex-growth`s to 0 and `flex-shrinks` to 1.

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
    </div>
  </div>
  <div class="footer">
    Footer
  </div>
</div>
```

If we view the layout now, we get the layout we want.

Suppose we put a lot of content on the main content. What do we think will happen to the sidebar? Will it remain short or will it stretch vertically to fill the parent
because main content's insides has more content?

```
...
<div class="main" fxFlex="4 0 10px">
  Main content
  <p>
    This is a paragraph to make this section taller.
  </p>
  <p>
    Here's another paragraph to make it even taller still.
  </p>
</div>
...

```

Let's run it again. Aha! The sidebar stretched to fill the height of the parent. This is an effect we've always wanted to have since we ditched tables for laying out
pages! Those of us who battled with page layout as of CSS2 found this feat extremely difficult, if not, impossible to pull. One dreaded hack was to create a graphic,
.. a GRAPHIC, that is so many pixels wide and 1 pixel high, with only two colors set it as a background image of the parent container (in our case, the content div),
and `repeat-y`. Another dreadful hack was to explicitly fix the heights with Javascript, and perhaps worse yet, jQuery. But with Flex Layout, this feat is practically
effortless to pull with just a few simple declarations in the markup!

It is obvious that our mock page isn't complete and is very trivial. The styling is created only to demonstrate how a few Flex Layout decorations can achieve the
desired placements and distributions of all of our containers.

## Next Steps

We will work off this page layout and make it responsive in the next article.