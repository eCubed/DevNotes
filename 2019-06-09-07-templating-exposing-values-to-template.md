# Angular Templating - Multiple Templates and Exposing Values To Templates

Asp.NET 4.x has the the Repeater control which allowed us to hand it an array, and it would take care of rendering the html *repeatedly* for each item
in the array and letting us decide the exact html markup we want for each item in a template we declare. It also let us know whether an item was an
alternating item with its alternating item template that it also let us define. In this article, we will implement the server-side Repeater control as
a component, with a few modifications and some additional features.

Let's first decide what exactly what we want to implement for our client-side repeater component:

1. It will take in an array of items, of any type. This component is merely a carrier of that array, and will not manipulate any of the items.
2. We will allow the implentor to specify an `itemTemplate` where they can decide the markup for each item to be rendered.
3. We will expose an `isAlternate` value to `itemTemplate` so that the implementor if they desire, may use it to change the appearance of the item if
the value of `isAlternate` is true.
4. We will allow the implementor to specify a `noItemsTemplate` that would show if the incoming array is empty.

## The Repeater Controller Code

```js
export class RepeaterComponent implements OnInit {

  @Input() items: Array<any>;

  @ContentChild('itemTemplate', {read: TemplateRef}) itemTemplate: TemplateRef<any>;
  @ContentChild('noItemsTemplate', {read: TemplateRef}) noItemsTemplate: TemplateRef<any>;

  constructor() { }

  ngOnInit() {
    this.items = (this.items == null) ? new Array<any>() : this.items;
  }
}

```

The controller code is pretty straightforward. We declare an @Input for the items array. We declare @ContentChild for `itemTemplate` and `noItemsTemplate`
so that the implementor can create custom markup for each item and for the case when there are no items. In the `ngOnInit`, we ensure that we have an
items array in case implementors didn't pass one in.

The work for the controller code is now complete.

## Repeater Markup

```html
<div class="repeater-container">
  <ng-container *ngIf="items.length > 0">
    <div class="repeater-item" *ngFor="let item of items, let i = index">
      <ng-container 
        *ngTemplateOutlet="itemTemplate; context: { $implicit: item, isAlternate: (i % 2 == 1)}">
      </ng-container>
    </div>
  </ng-container>
  <ng-container *ngIf="items.length == 0">
    <ng-container *ngTemplateOutlet="noItemsTemplate"></ng-container>
  </ng-container>
</div>
```

The code here contains many familiar directives and syntax, but may be put together in an unfamiliar manner. Let's start with the top-level. We wrap
everything in a single div, as good practice. Immediately underneath the top-level div are two `<ng-container>`s, one for when there are items to render,
and the second for when there are no items to render. As a review, we know that an `<ng-container>` is an element-less component we can use and apply
structural directives to so we don't end up having extra markup in the final render.

Let's focus first on the `noItemsTemplate`, since that is the easier of the two. We know that the contents of an `<ng-template>` whose identifier is
`noItemsTemplate` will be put in place of the `<ng-container>` inside the the `<ng-container *ngIf="items.length == 0">`

Now, let's focus on the *ngTemplateOutlet* for each item, which introduces a few new features.

First of all, we want an enclosing div for each item, and so we placed the *ngFor directive on that div. If we play close attention to the *ngFor's condition,
we also have `let i = index`. `index` is a value we have access to given by the *ngFor* directive for each item, and we just want to use the variable `i`
in its place for easier markup shortly.

Inside the item div is our `<ng-container>` that specifies the `itemTemplate` in its `ngTemplateOutlet` directive. Now, we are going to want to expose
an item of the array to the `<ng-template #itemTemplate>` up-front so that the implementor can display it in whatever markup needed. There is
the `context` object we will need to specify. The `context` object is a list of properties that will hold the values we will want to expose to the
`<ng-template>` up-front, and we need to take note of those properties because we'll need to refer to them later.

The `context` object allows us to specify the `$implicit` property, which is a special property meant to hold the value of the principal or primary object
we want to expose. In our case, `$implicit: item` means we're giving the value of `item` as stated in the `*ngFor` from our `items` array to the front
`<ng-template>` as the primary object to be rendered. After `$implicit`, we can go ahead and tack on other properties and give them values. In our case, 
we wanted to explose `isAlternate` to let the implementor know if the item's index is odd so it could be rendered differently (most commonly, change the
background color). So, we declare `isAlternate: (i % 2 == 1)`.

What we have left to do is to access those exposed values from the front now.

## Accessing Values Exposed to Template

We have previously discussed how to expose values from inside our templates-aware component to the templates at usage. We first present the full usage
of our template and then we'll discuss how the markup utilizes those exposed values:

```html
<lib-repeater [items]="items">
  <ng-template #itemTemplate let-item let-alternate="isAlternate">
    <div class="item" [class.alternate]="alternate">
      {{ item }}
    </div>
  </ng-template>
  <ng-template #noItemsTemplate>
    There are no items in the list!!!
  </ng-template>
</lib-repeater>
```

We didn't show the instantiation of the host component's `items` array, since we already know how that works. We also do not focus on `#noItemsTemplate`
since we have already discussed the basic value-less use of templating in the previous article.

Let's now focus on `<ng-template #itemTemplate>`. In the `<ng-template>` tag, we use `let-` to declare a variable that we can access from inside
that `<ng-template>`. The typical syntax is `let-variableNameWeWantUpFront`="variableNameExposedByNgTemplateOutlet". With `let-alternate="isAlternate"`,
we are accessing the `isAlternate` property as exposed from inside our component's `*ngTemplateOutlet` declaration, and "renaming" it to "alternate"
for use in our `<ng-template>`. Notice that the `let-item` isn't set to a specific exposed property. This is because we *already know* that we specified
a default/primary property via the `$implicit` property name back in `*ngTemplateOutlet`. We could've said `let-oneOfTheItems`, then `oneOfTheItems`
would hold the value of whatever we set `$implicit` to at `*ngTemplateOutlet`.

We didn't also specify the CSS rule for the class `.alternate` to keep our focus on templating. It will be left as a small exercise so that we can see
the effect of an exposed value in the final render. If we go ahead and render, provided that we declared a CSS class for `.alternate`, we will see that
the odd-indexed items will have a different background-color (or look).
