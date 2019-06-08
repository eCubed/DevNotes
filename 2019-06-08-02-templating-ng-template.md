# Angular Templating - the ng-template Tag

Angular has the `<ng-template>` tag that we can use in our components' markup:

```html
<ng-template>
    ng-template's contents. {{ variableFromTemplate }}
</ng-template>
```

If we try to render the page, we will notice that the contents of `<ng-template>` are *not* shown. We will specifically need to do something else to get
an `<ng-template>`'s contents to show, which we'll get into shortly.

## Rendering ng-template Contents When Conditions Call for its Displaying

One common thing that we would like to do is to show a list of items *only if* there are items to show in the first place, but when there are no items, we would
instead want to display a message. We can pull this with the following markup:

```html
<div class="list" *ngIf="items.length > 0">
  <div *ngFor="let item of items">
  </div>
</div>
<div class="no-items" *ngIf="items.length == 0">
  There are no items.
</div>
```

With the `<ng-template>`, we can write the following instead:

```html
<div class="list" *ngIf="items.length > 0; else #noItems">
  <div *ngFor="let item of items">
  </div>
</div>
<ng-template #noItems>
  There are no items.
</ng-template>
```
First, we give the `<ng-template>` an identifier so we can reference it. Then, in the *ngIf* of our list, we can add `else #ngTemplateIdentifier` which in our
case is `else #noItems`. This just says that we would display the template of that identifier when the condition in that *ngIf* is false. With the ng-template,
there is no longer the need to supply another *ngIf* statement on another element that is the opposite condition of the other element's *ngIf*.

This comes in really handy when we have several lists to display on the same component. All lists can use the same `#noItems` template if they have no contents.
This saves us from having to create a separate "no-items" div for each list, thus resulting in less markup to write and look at.

## Takeaways

The most important things in this article are (1) there is an `<ng-template>` tag that we can put in our components' markup, (2) its contents won't display
just yet until conditions are appropriate, and (3) we would identify it with `#templateIdentifier` in the markup so we can call upon it later so we can
finally display it.

## Next Steps

So far, we got the contents of `<ng-template>` to show in the place of an element when that element's *ngIf* condition is false. We can plop down `<ng-templates>`
on our page so that special kinds of components can access them and put them in appropriate places (or not have them show up at certain times).