# Flex Layout - Children Widths with Gaps

We will now explore setting widths on children components with gaps. We will still be working with the row layout. Our goal is for the children to
have equal widths, and a consistent gap between them, and we want them to fill their parent. We'll set the parent's padding to 10px so we can see 
that the children fill the parent.

## Flexible Equal Widths

```html
<div class="parent" fxLayout="row" fxLayoutGap="10px">
  <div class="child" fxFlex="1 0 10px">
    Item 1
  </div>
  <div class="child" fxFlex="1 0 10px">
    Item 2
  </div>
  <div class="child" fxFlex="1 0 10px">
    Item 3
  </div>
  <div class="child" fxFlex="1 0 10px">
    Item 4
  </div>
  <div class="child" fxFlex="1 0 10px">
    Item 5
  </div>
  <div class="child" fxFlex="1 0 10px">
    Item 6
  </div>
</div>
```
This is a case where we didn't want to calculate percentage widths for each item. All we want is for each item to grow into the same size such that
we would get a consistent gap between them, and that they will all fill their parent all the way. In order to pull this off, we gave each child the
same basis - a small one of 10px. We chose a small basis for two reasons. One, is that we want to override each child's "natural" width as defined by
its contents. Second, is we *intend* for each child *to grow* by its share of the amount of leftover horizontal space. Since each child's 
`flex-grow` is 1, they will all share the leftover horizontal space equally.

One particular interesting automatic calculation that happened is that Flex Layout took into consideration the `fxLayoutGap` when determining 
available free space. It knew that we have 6 items, and it meant 5 gutters of 10px each. So, the available space is P - 60px (combined basis of each
child) - 50px (5 gutters for 6 items at 10px each), which will be evenly distributed among the children.

This is *impossible* to pull in CSS, because in CSS, we *have to* specify a percentage width. If we added a 7th item, we would have to go back to our
CSS and change the percentage width of the child to fit them all, but we *would not need to change anything* in our Flex layout declarations! Also, 
it is worth mentioning that in CSS we would have had to set the width to `calc(0.16777% - 10px)` and set `margin-right` to 10px, but we wouldn't
want the 10px on the right-most item. Either we would use a `:not(:last-child)` selector so that the last child won't get the `margin-right` rule,
or we set the parent's padding-right to 0 because the last child's right-margin would act as the parent's padding. All that hacky styling is not
necessary in Angular Flex Layout.

## Even Width with Percentages and `space-between` Alignment

The previous example will only work if we intend to have one row of items. In many applications like an image gallery, we do want equally-wide
children, a consistent gap between them, but also to wrap to the next line when the next child no longer fits the remaining horizontal space.

```html
<div class="parent" fxLayout="row wrap" fxLayoutAlign="space-between">
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 1
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 2
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 3
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 4
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 5
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 6
  </div>
</div>
```
The approach here will need to be different than the previous example. First of all, we added `wrap` to `fxLayout` so that additional content
that won't fit in a row will "move to the next line". We also did *not* specify a `fxLayoutGap` declaration, but instead we specified
`fxLayoutAlign="space-between"`. Since we're wrapping content now, the `space-between` setting will ensure that all of the items in a single row
before that next item that won't fit in that line, will fill the entire width of the parent, and automatically create a consistent space between
children.

We are setting the basis with a calculation - `calc(25% - 10px)`. The `25%` will give us the 4 columns we want, and the `10px` is the gutter we'd
want to subtract from `25%` so that each column is a bit less than `25%` of the actual parent's width. This calculation sets the actual width of the
4 columns with gutter in mind, and we'll let `fxLayoutAlign="space-between" do the job of distributing the children horizontally. Since we
already calculated the gutter into the child width, we'll end up with the exact gutter we want after `space-between` does its job. Note
that we set `flex-grow` to 0 because we don't want our children's widths to grow no matter what. We set the `flex-shrink` to 1 in case the
calculation, due to rounding errors, may cause the children's total width to exceed the parent's width. In this case, we would want to shrink each 
child equally to fit 100% of their parent's width.

There are a couple of problems, though. One, is that the children have no `margin-bottom`. Let us recall that since our layout is row, though 
wrapped, Flex Layout, and even CSS Flexbox will only apply a `margin-right`. If we want, we can go into the child's CSS and set `margin-right: 
10px`. This would fix the space between lines issue. The larger problem is the effect of `space-between` when the last line of items from the
wrapping has less items than there are columns. In our example, we have 6 items in our attempt to arrange them in 4 columns. The second and last
line has two items, but one is all the way to the left, and the other is all the way to the right. It doesn't look very pleasing. We would've like
item 6 to be right underneath item 2. This is expected behavior because `space-between` adds necessary space between elements to fill the parent, 
and if there are only 2, we'll get that one huge space in the middle. The easy "cure" for this is to ensure that the number of items must be evenly
divisible by the number of columns. This isn't always a guarantee, though!

## Even Width Percentages with `fxLayoutGap`

The previous example works well with each line automatically filling each row, but looks tacky when the last line doesn't have enough items to fill 
all columns. In this next example, we'll attempt to create guttered columns, and this time, we want to "justify left" the fewer-than-number-of-columns
items in the last line.

```html
<div class="parent" fxLayout="row wrap" fxLayoutGap="10px">
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 1
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 2
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 3
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 4
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 5
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 6
  </div>
</div>
```
We *did* get the "left-justify" effect we wanted, however, there is extra padding on the right-hand side of the container - 10px for the parent's
own padding, and 10px `margin-right` for the last item in the first line. But this approach is the same as the first example, though, so why did
we get the extra padding? Recall that `fxLayoutGap` will only *not give* the `margin-right` to the *very last child*. The very last child is Item
6, not Item 4, which is only at the end of the first line. Since Item 4 isn't the absolute last child, it *still* gets a margin-right.

So, how do we cure this? Unfortunately, we'll need to hack at the parent to set its right-padding to zero. We created a new CSS class,
`no-right-padding` which has only one rule - `padding-right: 0;`, and applied it to the parent container.

```html
<div class="parent no-right-padding" fxLayout="row wrap" fxLayoutGap="10px">
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 1
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 2
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 3
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 4
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 5
  </div>
  <div class="child" fxFlex="0 1 calc(25% - 10px)">
    Item 6
  </div>
</div>
```
Now, if we view our container, we'll see exactly what we want - even columns with gutter, and left-justified last line if it didn't have as many
items as there are columns.

## Afterthoughts

So far, we have seen how easy it is to make Flex Layout declarations in markup as opposed to hacking away at CSS with floating and clearing divs, and
trading off parent padding for an edge child's margin. We still performed a little bit of parent padding vs. child margin tradeoff, but it achieved
a very common desire when laying out items of equal widths (and equal heights).

One objection that may come up is that we're putting back some physical features in our html. We see explicit units of measure in a lot of the Flex
Layout directives, and even a CSS `calc` computation. Flex Layout's goal is for us NOT to write CSS *for layout*, but instead declare them in markup.
We may still write CSS classes, as we did here, but *only* for colors, fonts, etc. At least Flex Layout declarations are *not* as messy and unmaintainable
as actual inline CSS styles, and we still should avoid those. But the small amount of "jarring layout specifications" in the markup is just a small
penalty that solves some common CSS problems that are tough to pull in plain CSS, though! When we get to responsive design with Flex Layout, we'll
see how easy it is to declare, which would have resulted in tons of confusing and eye-straining media queries in CSS.


