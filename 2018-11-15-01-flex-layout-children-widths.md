# Flex Layout - Children Widths

So far, we have *not* explicitly declared children widths. We assumed whatever width they would come in depending on their content. In this article, we will explore how
to set children's widths. We shall assume that the parent's layout specifies `row` so that we'll be manipulating children's widths.

## The `fxFlex` Directive

We can apply `fxFlex` on child elements in order to specify their widths. Be forewarned that declaring widths is *not* exactly as straight-forward as setting a percentage
value and calling it a day.

The full form of `fxFlex` is `fxFlex="<flex-grow> <flex-shrink> <flex-basis>"`, and an example would be `fxFlex="1 0 130px"`. These correspond to `flex-grow`,
`flex-shrink`, and `flex-basis` from CSS Flexbox. Now, let's explain what these three properties do.

* **flex-basis**. The starting width value of the child container. Its value must have a unit like percent, em, or px. Another possible value is `auto` which means take
whatever width it may be as driven by its contents.

* **flex-grow**. This is an arbitrary unit-less number greater-than or equal to zero which indicates ultimately how much to increase the element's width by, if the parent
has extra leftover space. For now, a value of 0 means the element will not grow no matter what - it will stay the same width, and a value greater than 0 means the element will grow by some amount.

* **flex-shrink**. This is an arbitrary unit-less number greater-than or equal to zero which indicates ultimately how much to decrease the element's width by, if
the siblings' total width exceeds the parent's width. For now, a value of 0 means the element will not shrink no matter what - it will stay the same width, and a value
greater than 0 means that it will shrink by some amount.

Be aware that the values for `flex-grow` and `flex-shrink`, especially `flex-grow`, are *not* proportional values of sibling elements' final widths relative to each
other! This is a very common mistake that many tutorials don't really clarify, thus is a very confusing point for designers and developers.

## Setup CSS

We will style child and parent so we can see what's going on clearly when we run the app.

```css
.parent {
  background-color: #ccc;
  width: 1000px;
}

.child {
  box-sizing: border-box;
  padding: 10px;
  background-color: #ffb039;
  border: solid 1px #d78100;
}
```

In all of the following examples, the parent's with has been set to 1000px in the CSS so that it becomes easier to calculate values when we're analyzing 
how grow, shrink, and basis values determine the ultimate width of each child.

## No `fxFlex` Specified - Default

We'll start really easy with a parent container with only one content where we don't specify any `fxFlex`:

```html
<div class="parent" fxLayout="row">
  <div class="child">
    No fxFlex specified
  </div>
</div>
```
Without specifying fxFlex, the default value, in (grow, shrink, basis) "coordinates" is 0 1 auto. So, 0 for grow means don't grow - take whatever the width
is of the child. Now, the shrink 1 means we will shrink the child by some amount, but only if there is not enough horizontal space to contain all the
siblings.

## Specifying `flex-basis` with an Absolute Pixel Value

```html
<div class="parent" fxLayout="row">
  <div class="child" fxFlex="300px">
    fxFlex="300px"
  </div>
</div>
```
When fxFlex has 1 parameter specified, and that parameter is a value *with a specified unit*, it automatically *is* the basis value. This is the equivalent 
of "0 1 300px", since 0 and 1 are the defaults for grow and shrink parameters, respectively. Basis values may also be specified in percentage or em. For now, 
we'll stick with absolute pixels for analysis when we need to investigate unexpected results.

## Two children with `flex-basis` of Absolute Pixel Values

```html
<div class="parent" fxLayout="row">
  <div class="child" fxFlex="300px">
    fxFlex="300px"
  </div>
  <div class="child" fxFlex="500px">
    fxFlex="500px"
  </div>
</div>
```
In this example, we have two children which we specify explicit widths. The children will simply display as expected with the widths specified.

## Two children with `flex-basis` and `flex-grow` Specified

```html
<div class="parent" fxLayout="row">
  <div class="child" fxFlex="1 0 300px">
    fxFlex="1 0 300px"
  </div>
  <div class="child" fxFlex="1 0 500px">
    fxFlex="1 0 500px"
  </div>
</div>
```
In this example, we have two children which we specify explicit widths. We also allowed each container to grow by setting the first parameter to 1. We all 
know that flex-grow and flex-shrink values are *unitless*, thus might represent proportional values. Since we set the flex-grow equal to 1 for both children,
we'll get the impression that they are both ultimately going to end up having equal widths. Alas, this is a false assumption, though logical. Many tutorials
will even say that the value is a "growth factor", which it's not! We'll see why it's not a growth factor as we go through further examples.

Flex growth works like this. First, we'll need to calculate the leftover space, if any. That's 1000px for the parent minus 800px (the total of the widths of 
the two children), which leaves us with 200px. Now, This 200px is what we'll *distribute* across the children *proportionally*, and that's where the 
flex-growth values come into play. We've got 1 for both of them, so intuitively, they both get equal share of the 200px. This will make each grow by 100px
*in addition* to the basis values of 300px and 500px that we initially set. The new widths are now 400px and 600px, totalling 1000px, which would fill the
parent.

We discussed earlier that `flex-growth` isn't a "growth-factor". The term "factor" indicates that there a certain constant number is to be multiplied to
a base number to get the final value. This is *not* what's happening *at all*. For one, we could not have multiplied the basis width by 1 to get a larger
basis. As calculated above, the way to get to a child's final width with `flex-growth` is *not* simply multiplying by a number!

Before we proceed, there is a disclaimer about the `flex-shrink` value. Notice that we've set it to zero. This is what we need to do for Angular Flex Layout
where the 1 would have been fine for regular CSS. Under regular circumstances, the 1 would  be OK because the total starting widths total less than the 
parent's width, which would not activate flex-shrink. For some mysterious reason, Angular Flex Layout will only let us either shrink or grow, one or the 
other, not both, so one of those values always has to be 0. If both flex-grow and flex-shrink are greater than zero, we will not achieve the desired results.

## Two Children whose `flex-grow` are Not Equal

```html
<div class="parent" fxLayout="row">
  <div class="child" fxFlex="3 0 300px">
    fxFlex="3 0 300px"
  </div>
  <div class="child" fxFlex="1 0 500px">
    fxFlex="1 0 500px"
  </div>
</div>
```
We've given the first child a `flex-grow` value of 3 and the other child a `flex-grow` value of 1. First impression says that the final width of the first
child will be 3x the final width of the other child. If we look at the results, that assumption is way off. Actually, the first child turned out to be 
slightly narrower than the other one. How did this happen? We have a left-over space of 200px. Now, we will need to share these 200px between the two 
children, but now, how much will each get? The flex-grow values 3 and 1 mean that first child will get 3x as much *of the leftover* than the other child.
There is a total of 4 flex-grow values among the siblings. So, the first child gets 3 out of 4 of the 200px, which is 3/4 * 200px = 150px. So, its final 
width would be 300px original + 150px from the flex-grow calculation, which is 450px. Now, for the second child, it only got 1 out of 4 of the 200px, which 
is 1/4 * 200px, or 50px. So, the final width of that second child is 500px basis + 50px from the flex-growth calculation, or 550px.

## Letting the Last Child Fill the Remaining Space

A common scenario is when we want to lay out a page where the left-hand sidebar's width might need to be exactly some pixels wide, and the content should
fill the rest of the parent's width. This would be quite tough to pull in regular CSS, not to mention very hacky and unpredictable.

```html
<div class="parent" fxLayout="row">
  <div class="child" fxFlex="0 0 300px">
    fxFlex="0 0 300px"
  </div>
  <div class="child" fxFlex="1 0 10px">
    fxFlex="1 0 10px"
  </div>
</div>
```
Here, we have a situation where we might want the first child to be a specific width, and then we want the next and only child to fill up the rest of the 
space. For the first child, we set flex-grow and flex-shrink to be both 0 because we want it to always be 300px no matter what. We know that the parent is
1000px, so we might as well would have set fxFlex for the second child to be "0 0 700px". But we won't always know the parent's width, so we need to rely on
the `flex-grow` values.

Recall that flex-grow comes from *the leftover space* if the parent is still wider than the width of its children combined. The first child's flex-grow value
is 0, so it won't grow. The flex-grow value of the second child is 1. There is a total of 1 flex-grow values among the children, but all of it is for the 
second child. This means that the second child will get *all* of the leftover pixels. Leftover pixels is 1000px (parent) - 310px (sum of the children's basis
widths) = 690px. This means that the   second child will get all of the leftover 690px in addition to the initial 10px, which total 700px. This kind of 
example shows that we can't fully rely on the starting widths we specified.

## The Spacer

A common scenario we may want to pull is to butt up a child to the left-hand side of the container, and another child to the right-hand side of the container. 
This is very useful when we lay out toolbars where the logo and company name usually go to the left, and the navigation links usually go to the right, and 
there is a space in the middle. With regular CSS, this is very tough, hacky, and unpredictable to achieve. In this case, we won't know the widths of the 
two children on either side. Let's also pretend that we don't know the parent's width, though we know it's actually 1000px in our case. 
So, we're going to write everything in algebraic terms.

```html
<div class="parent" fxLayout="row">
  <div class="child" fxFlex="0 0 auto">
    fxFlex="0 0 auto"
  </div>
  <div class="child" fxFlex="1 0 auto">
    Filler: fxFlex="1 0 auto"
  </div>
  <div class="child" fxFlex="0 0 auto">
    fxFlex="0 0 auto"
  </div>
</div>
```
We'll let X be the width of the first child, Y be the width of the spacer, and Z be the width of the third child. Also, let P be the width of the parent. So,
the leftover width is P - X - Y - Z. Now, let's look at the `flex-grow` values. The first and third child have `flex-grow` 0, so they're going to remain 
whatever size they are. The total `flex-grow` values among the three children is 1, and the spacer child's flex-grow is 1, so it gets all of P - X - Y - Z.
So, finally, that spacer child's final width is Y + P - X - Y - Z.  Ys cancel, so we're left with P - X - Z. Since the children on opposite sides don't get 
any of that leftover space because their `flex-grow` values are 0, their final widths are still X and Z. So, let's add up all the final widths, and they 
should end up equalling P, or the parent's total width. X + Z + P - X - Z = P. Indeed!

The result is that the first and last children will be pushed to the sides, and we have that child in between them that would will the remaining space.

## Shrinking the Children to Fill Parent, Equal `flex-shrink` values
We saw in the previous examples how `flex-grow` actually works. Now, let's explore what `flex-shrink` does.

```html
<div class="parent" fxLayout="row">
  <div class="child" fxFlex="0 1 500px">
    fxFlex="0 1 500px"
  </div>
  <div class="child" fxFlex="0 1 700px">
    fxFlex="0 1 700px"
  </div>
</div>
```
`flex-shrink` works a similar way as `flex-grow`, however does the opposite. Instead of adding width to fill their parent as `flex-grow` does, `flex-fill` 
*chops off* width from the children so that they can fit inside and fill their parent. In this example, we see that we get an overflow of 200px. We gave both
children a `flex-shrink` of 1. This means that they'll equally split that 200px overflow, and each of their shares of the overflow will be chopped off from
their original widths. So, 100px will be chopped off the first child, resulting in 400px, and 100px will be chopped off the second child, resulting in 600px.
Now, the new widths of 400px and 600px add up to 1000px, the parent's width.

## Shrinking the Children to Fill Parent, Unequal `flex-shrink` Values

```html
<div class="parent" fxLayout="row">
  <div class="child" fxFlex="0 9 500px">
    fxFlex="0 9 500px"
  </div>
  <div class="child" fxFlex="0 1 700px">
    fxFlex="0 1 700px"
  </div>
</div>
```
Now, we have an overflow situation where the `flex-shrink` values are not the same. Again, the values might give the impression that the final width
of the first child will be 9x the width of the second child. Just like in the `flex-grow` situation, that assumption is false. We've got an overflow
of 200px. This means that a total of 200px will be chopped off with a different amount off each child, but we now need to figure out how much each
child will be chopped off. There are a total of 10 `flex-shrink` units. The first child gets 9/10 of the 200px chopped off it, or 180px. So, its final
width is 500px (original) - 180px, which is 320px. The second child gets 1/10 of the 200px chopped off it, or 10px. Its final width will then be 700px
(original) - 10px, wich is 680px. So, the new widths 320px + 680x total 1000px, which they now fit and fill their parent. Despite the high-looking 
`flex-shrink` 9 given to the first child, it still ended up being narrower than the second child, which confirms that `flex-shrink`, and also 
`flex-grow` values are *not* weights of the final widths of the children relative to each other to fill their parent!

## Next Steps

We've thoroughly investigated how `flex-grow`, `flex-shrink`, and `flex-basis` work, and demystified one of CSS's most confusing topics. In all of
these examples, we have not thrown in `flexLayoutGap` in the mix. This will affect what we'll put in children's `fxFlex` directives. Right now, if we specify a `flexLayoutGap` for the parent, and leave the `flex-basis` values as they are, even when they add up to
100%, we'll end up with an overflow. We'll investigate this in a later article.

We may also present more practical articles that actually easily solve some layout problems we've had since we first got our hands on CSS 2, using
Flex Layout.

We will also explore one of Angular Flex Layout's most powerful features, and that is the ability to specify `fx` declarations for various screen widths.

