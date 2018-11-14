# Flex Layout - Basics

In this article, we will see some basic Flex Layout declarations in action. We will identify a parent div and its immediate children, who are also divs, with only this single-level
hierarchy.

We start with a component that lists names of cities, and we are to lay them out using Flex Layout.

```javascript
export class ParentChildComponent implements OnInit {

  cities: Array<string>;

  constructor() { }

  ngOnInit() {
    this.cities = [
      'Chicago',
      'Los Angeles',
      'New York',
      'Paris',
      'Tokyo'
    ]
  }
}
```

The starting markup:

```html
<div class="cities-list">
  <div class="city-item" *ngFor="let city of cities">
    {{ city }}
  </div>
</div>
```

The styling:

```css
.cities-list {
  background-color: #cccccc;
  padding: 10px;
}

.city-item {
  background-color: #9dbbff;
  border: solid 1px #3124f1;
  padding: 10px;
}
```

At this point, we have not yet put a single Flex Layout directive in the markup. We're still good with our styling because we've only specified physical appearance, not layout. When
we view the page right now, we will see the cities vertically stacked on top of each other because that's the default div behavior - fill up all of the horizontal space because it's
a block. We don't see the gray background because the children fill their parent.

## The `fxLayout` Directive

`fxLayout` is an attribute directive that pertains to the parent container. In Flex Layout, a parent's children can only be laid out in a single column or a single row. There is no
such thing as a layout grid in Flex Layout. So, let's see what happens when we want to lay out the cities side-by-side:

```html
<div class="cities-list" fxLayout="row">
  <div class="city-item" *ngFor="let city of cities">
    {{ city }}
  </div>
</div>
```

We'll see that the sibling elements are now side-by-side without gaps. Each child will only take the horizontal space it needs, which may leave an empty space. To pull this in CSS would
have required floating and a clearing-div hack.

## The `fxLayoutGap` Directive

What if we wanted a consistent pixel-width gap between each item? Normally, in CSS, we would specify it on the child to have a `margin-right` rule for this. In Flex Layout, this is
declared in the parent with `fxLayoutGap`.

```html
<div class="cities-list" fxLayout="row" fxLayoutGap="10px">
  <div class="city-item" *ngFor="let city of cities">
    {{ city }}
  </div>
</div>
```

We will now see that there is a 10px gap between the children. Gap is a layout specification, so we don't specify it in CSS.

Just to play around, what if we kept `fxLayoutGap` intact, and we change `fxLayout` to column?

```html
<div class="cities-list" fxLayout="column" fxLayoutGap="10px">
  <div class="city-item" *ngFor="let city of cities">
    {{ city }}
  </div>
</div>
```

The `fxLayoutGap` would create the effect of a `margin-bottom` rule, but *not* for the last child. This is more often than not the effect we want, especially if we specified
a padding for the parent. In plain CSS, we would have needed to specify `margin-bottom` for the children. When we do this, we will see a thick padding on the bottom of the
container, which accounts for both the padding of the child, and the padding of the parent. To make the padding uniform on the parent, it will have to give up its `padding-bottom`
and make that last child's `margin-bottom` to *act like* the entire container's `padding-bottom`. That certainly is doable but it feels extremely goofy.

So, in summary, `fxLayoutGap`, though declared on the parent, will act like `margin-right` for each child if `fxLayout` is `row`, and will act as `margin-bottom` for
each child if `fxLayout` is `column`.

## Wrapping

Wrapping pertains only to,`fxLayout` `row` since it addresses what would happen if the leftover horizontal space of the parent container is too small for the next child to fit.
Flex Layout allows us to specify `wrap` in conjuction with `row` on `fxLayout`. We added more cities to the list to show the effect. If we ran the app right now without specifying
the wrap, and we shrink the browser's width, we will see that the children will overflow endlessly to the right of their parent's edge.

However, if we specify `wrap`, there effect would be somewhat like a line break. When it doesn't fit the leftover horizontal space, "go to the next line".

```html
<div class="cities-list" fxLayout="row wrap" fxLayoutGap="10px">
  <div class="city-item" *ngFor="let city of cities">
    {{ city }}
  </div>
</div>
```

One thing to note, though, that there *isn't* a `margin-bottom` on each child when a wrap happens. This is because `fxLayout` is set to `row`, and will *only* simulate a
`margin-right`. If we want the padding between each row caused by wrapping, we will need to pull some CSS tricks, which again, are unintuitive and goofy. Nevertheless, wrapping
is good to know, and may have its place in laying out inline-block-like elements.

## Next Steps

We will still be exploring a single-hierarchy parent-children markup, so we will explore next the various ways to spatially distribute children inside their parent. Concerns to
address would be how to create a consistent gap between each child so that they'll all fill the parent's width. Another is how to vertically or horizontally align them inside
their parent.

After we've explored spatial children distribution, then we can look at how to force each child to have uniform widths.

We have already seen how little we need to specify Flex Layout declarations in markup when their plain CSS equivalents are longer and somewhat contortionist to write and maintain.
A bit later, we'll explore how to declare specifications that would only apply to certain browser widths for a responsive experience. It is a little bit more markup code to write,
of course, but that small addition to markup code is worth even a ton more confusing and eye-straining CSS code that we no longer have to write and maintain.


