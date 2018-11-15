# Flex Layout - Alignment

So far, we've been able to declare the parent container's layout with `fxLayout`, meaning whether the children will be displayed horizontally (`row`)
or vertically (`column`). We were also able to wrap content when the parent's `fxLayout` specifies `wrap`, and also set specify `fxLayoutGap` which
is the space between children in either layout direction.

We notice that the children in our parent container are always left-aligned. It is possible to align them right, or center, or others.

## Horizontally Aligning Children with `fxLayoutAlign`

By default, or without specifying alignment, children of a Flex Layout container will be left-aligned, which is equivalent to `fxLayoutAlign="start"`.
There are possible values for `fxLayoutAlign` for horizontal alignment. We'll discuss them below:

* **start** - aligns children to the left inside their parent.
* **center** - center aligns children inside their parent.
* **end** - right-aligns children inside their parent.
* **space-between** - the first and last items are butted up against the left and right edges, respectively of the parent. All other children between first
and last are evenly distributed in the middle with the same horizontal space width between each child element.
* **space-around** - there is a consistent space width between each element, however, the first element gets a left margin half of that space width and the
last element gets a right margin half of that space width. It is almost the same as space-between, except we don't butt up the first and last elements onto
either left or right edges of the parent.

Here is an example of a parent where its children will be horizontally aligned at its center:

```html
<div class="cities-list" fxLayout="row" fxLayoutAlign="center">
  <div class="city-item" *ngFor="let city of cities">
    {{ city }}
  </div>
</div>
```

## Vertically Aligning Children with `fxLayoutAlign`

We can also vertically-align children inside their parent container. However, if we do, there are a few requirements:

* The parent container's height must be explicitly set either in inline css (which we shouldn't do) or an actual style-class. If not done, then there is
no way for the browser to calculate vertical position for each child.
* We MUST first specify horizontal alignment! Then to specify vertical alignment, we type a single space, then a possible vertical align value.

So, we'll introduce a new css class that specifies a fixed height:

```css
.cities-list-tall {
  background-color: #ffbd70;
  height: 150px;
  padding: 10px;
}
```

There are a few possible values for vertical alignment of children in their parent, and they are the *second parameter* of `fxLayoutAlign`

* **start** - the children's top edge will butt up against the top edge of their parent.
* **center** - Each child, since each may have a different height, will have equal space between their top edge and the parent's top, and between their
bottom edge and the parent's bottom edge.
* **end** - the children's bottom edge will butt up against the bottom edge of their parent.
* **stretch** - the children's vertical bounds (top and bottom) will fill the parent.

Note that `space-between` and `space-around` does not pertain to vertical alignment!

Here is an example of a container with a fixed height, and whose children are left-aligned, and also bottom-aligned.

```html
<h3>Basic Row Layout - Vertical Align Bottom (End)</h3>
<div class="cities-list-tall" fxLayout="row wrap" fxLayoutGap="10px"
  fxLayoutAlign="start end">
  <div class="city-item" *ngFor="let city of cities">
    {{ city }}
  </div>
</div>
```

## Alignment when `fxLayout` is `columnn`

We will leave this up to the reader to toy around with. Typically, in web design practice, we minimize explicitly setting the height of a container so we could align
its children along the horizontal axis. We typically specify widths only because content ultimately determine the height of their parent containers. However, it is possible
to align when `fxLayout="column"`, though we don't recommend it.

## Next Steps

We've aligned children as a group inside their parent. We've aligned them left, right, center, and distribute evenly with `space-around` and `space-between`. So far,
though, we have let each child be as wide as it need to be, according to its content. We will next explore how to properly assign widths to children elements. This will
greatly help in laying out main rectangular sections on the screen.



