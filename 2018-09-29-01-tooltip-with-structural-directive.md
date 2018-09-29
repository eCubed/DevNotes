# Tooltip with Structural Directives

We will attempt to create a tooltip directive where the user would hover over the element or component, and then
a rectangular area would pop up containing text or templated content. Then the popup would disappear when they
mouse-leave.

Eventually, we would like to apply our `libTooltip` directive this way:

```html
<p>
  The quick brown fox jumps over the head of the lazy
  <span *libTooltip="'which is a St. Bernard'">dog</span>
</p>
```
or

```html
<img src="..." *libTooltip*="imageTooltipTemplate"></img>

<ng-template #imageTooltip>
  <div>
    This is what I want to show in the popup. Stuff about the image.
  </div>
</ng-template>
```

## Starting with the Tooltip Component

Since we will need to wrap html around a structural directive's template, we will need to create a container component.
However, we will need
