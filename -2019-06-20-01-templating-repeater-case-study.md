# Angular Templating - Repeater Case Study

Asp.NET 4.x has the the Repeater control which allowed us to hand it an array, and it would take care of rendering the html *repeatedly* for each item
in the array and letting us decide the exact html markup we want for each item in a template we declare. It also let us know whether an item was an
alternating item with its alternating item template that it also let us define. In this article, we will implement the server-side Repeater control as
a component, with a few modifications and some additional features.

Let's first decide what exactly what we want to implement for our client-side repeater component:

1. It will take in an array of items, of any type. This component is merely a carrier of that array, and will not manipulate any of the items.
2. We will allow the implentor to specify an `itemTemplate` where they can decide the markup for each item to be rendered. We would also specify a
`defaultItemTemplate` should they omit `itemTemplate` from the front.
3. We will expose an `isAlternate` value to `itemTemplate` so that the implementor if they desire, may use it to change the appearance of the item if
the value of `isAlternate` is true.
4. We will allow a `layoutTemplate` so that our component wouldn't hold our items and no items template hostage to a div element. We'll also specify
`defaultLayoutTemplate`

## The Starting Repeater Component Code

Let us start declaring the properties for all of our templates and placeholders in our markup as well as the `@Input()` for our `items` array of type `any`:

```js
export class RepeaterComponent implements OnInit {

  @Input() items: Array<any>;

  @ContentChild('layoutTemplate', { read: TemplateRef }) layoutTemplate: TemplateRef<any>;
  @ViewChild('defaultLayoutTemplate',  { read: TemplateRef }) defaultLayoutTemplate: TemplateRef<any>;

  @ContentChild('itemTemplate', { read: TemplateRef }) itemTemplate: TemplateRef<any>;
  @ViewChild('defaultItemTemplate', { read: TemplateRef }) defaultItemTemplate: TemplateRef<any>;

  @ContentChildren(TemplatePlaceholderComponent) templatePlaceholdersQueryList: QueryList<TemplatePlaceholderComponent>;

  itemsPlaceholder: TemplatePlaceholderComponent;

  constructor(
  ) { }

  ...
}

```
As a review, we create `@ContentChild` for templates we would expect from the front, and the `@ViewChild` for their corresponding default templates.

In the case where the implementor specifies a custom `#layoutTemplate` where we'll need to hunt for the `itemsPlaceholder`, we will need the
`@ContentChildren` `QueryList` that would get us a queryable list of all components that end up being a part of our component that are of type
`TemplatePlaceholder`.

## Repeater Markup

```html
<ng-container *ngTemplateOutlet="layoutTemplate || defaultLayoutTemplate">
</ng-container>

<ng-template #defaultLayoutTemplate>
  <div>
    <div>Default Layout Template for Repeater</div>
    <div>
      <ng-container *ngFor="let item of items, let i = index">
        <ng-container *ngTemplateOutlet="itemTemplate || defaultItemTemplate; context: createItemTemplateContext(item, i)"></ng-container>
      </ng-container>
    </div>
  </div>
</ng-template>

<ng-template #defaultItemTemplate let-item let-isAlternate="isAlternate">
  <div>
    {{ item | json }} (Providing custom itemTemplate is encouraged)
  </div>
</ng-template>
```

The general structure of our markup is quite similar to that of our Panel component's, but of course with differences having to do with this panel's different
purpose, which is to display items one at a time, one after another, using a template (user-provided or default) to render it.

Notice in the `#defaultLayoutTemplate`, we have an `<ng-container>` to which we would apply the familiar `*ngFor` directive. This way, each item wouldn't
be "held hostage" by an html element that our component decides. The `*ngFor` also gives us the index of the item being rendered, which we'll need to use later
to determine whether an item is the alternate item or not.

There are a few things we would want to point out with the `#defaultItemTemplate`. The first one is that we decided to not style it and merely display the
item's JSON. If that object has tons of properties, then the display would be very crowded. We then put an explicit message on there encouraging the impleentor
to specify a custom `#itemTemplate` instead. The second thing is the value-less attribute `let-item`. Notice that we didn't specify what property from the
template context to reference and regard as our local variable `item`. We will address how that became possible.

Right now, our component's markup is done. The remainder of the work is handling in code the case where the user does specify a `#layoutTemplate`.

## Code Managing Layout and Placeholders

First, we'll put the skeleton code and we'll fill it up as we go along.

```javascript
export class RepeaterComponent implements OnInit, AfterContentInit, DoCheck {

  @Input() items: Array<any>;

  @ContentChild('layoutTemplate', { read: TemplateRef }) layoutTemplate: TemplateRef<any>;
  @ViewChild('defaultLayoutTemplate',  { read: TemplateRef }) defaultLayoutTemplate: TemplateRef<any>;

  @ContentChild('itemTemplate', { read: TemplateRef }) itemTemplate: TemplateRef<any>;
  @ViewChild('defaultItemTemplate', { read: TemplateRef }) defaultItemTemplate: TemplateRef<any>;

  @ContentChild('noItemsTemplate', { read: TemplateRef }) noItemsTemplate: TemplateRef<any>;

  @ContentChildren(TemplatePlaceholderComponent) templatePlaceholdersQueryList: QueryList<TemplatePlaceholderComponent>;

  itemsPlaceholder: TemplatePlaceholderComponent;

  constructor(
  ) { }

  ngOnInit() {
    this.items = (this.items == null) ? new Array<any>() : this.items;
    setTimeout(() => { this.ngDoCheck(); }, 0);
  }

  createItemTemplateContext(item: any, index: number) {
    return {
      ...
    };
  }

  private buildTemplate() {
    if (this.itemsPlaceholder != null) {
      ...
    }
  }

  ngDoCheck() {
    if (this.itemsPlaceholder != null) {
      this.buildTemplate();
    }
  }

  ngAfterContentInit() {
    this.templatePlaceholdersQueryList.changes.subscribe(c => {
      ...
      this.buildTemplate();
    });
  }  

  ...
```

First, the `ngOnInit()` is called, and in that life cycle function, our data from `@Input() items` is available, if provided. If not, it'll just be an
empty array.

Right after `ngOnInit()`, the `ngDoCheck()` is called. This is when we'll need to populate the `itemsPlaceholder` programmatically, but early on in the
component's life cycle, the `itemsPlaceholder` is null because at this point, the template contents aren't available yet. So, we null-guard `itemsPlaceholder`.
We also bear in mind that this `ngDoCheck()` function will be called every time Angular detects any changes to `items`, like new item found or an item got
deleted in the `@Input() items`.

We now head to `ngAfterContentInit()`, a function that would be called later in the component's life, when the external templates have been found and loaded
to our component. This is when the `templatePlaceholdersQueryList` becomes available and we would still need to subscribe to its changes event to get to the
point where we're sure that the all components declared inside the template from up-front, have finally made it into our controller.

## Building Each Item Programmatically

Now that all of the template components have finally loaded, we can now query the `templatePlaceholdersQueryList` for a specific `TemplatePlaceholder` by
id, and save it (if found) to `this.itemsPlaceholder`:

```javascript
  ngAfterContentInit() {
    this.ltemplatePlaceholdersQueryList.changes.subscribe(c => {
      this.itemsPlaceholder = this.templatePlaceholdersQueryList.find(i => i.id === 'itemsPlaceholder');
      this.buildTemplate();
    });
  }
```

Now that we found the `itemsPlaceholder`, we can now programmatically build the UI for each item:

```javascript
  private buildTemplate() {
    if (this.itemsPlaceholder != null) {
      this.itemsPlaceholder.clear();

      this.items.forEach((item, index) => {
        this.itemsPlaceholder.append(this.itemTemplate || this.defaultItemTemplate, this.createItemTemplateContext(item, index), index);
      });
    }
  }
```

Note that we *always* clear the placeholder first before adding items to it. Otherwise, Angular will just append to the existing itens, which is surely NOT what
we want to happen! We iterate over each item, and for each one, we call `itemPlaceHolder.append(...)`. Note that the `.append()` function asks for 3
arguments - in order, the template, the context, and the index. Since what we're rendering are array items, we'll want to pass the index since the `ViewContainerRef`
inside the place holder asks for an index.

We are left with implementing the `.createItemTemplateContext(item, index)`, which we hand over the ite itself and then the index so we could determine
if the item is alternate or not.

```javascript
  createItemTemplateContext(item: any, index: number) {
    return {
      $implicit: item,
      isAlternate: (index % 2 === 1),
      index: index
    };
  }
```

We expose the property `isAlternate`, which we decided to be the odd-indexed entries with the determining condition `(index % 2 === 1)`, and the index in case
the user wants to use that value for some unique display decisions. Then we expose the item itself with the special property `$implicit` that a template context
recognizes. We could have said `{ item: item, ...}` instead, but at the front, that would require us to access the item from the context with `let-item="item"`.
The `$implicit` property would let us omit referencing a specific property from the template context, which lets us merely declare `let-item` in the template.
We can regard `$implicit` to be the leading, main, or principal object we want to expose to an `<ng-template>`.

## Further Discussions and Exploration

Why would we even bother to create a Repeater component in the first place if all it does is render an array of items? Couldn't we do that in-component with a
regular in-hosting-component `*ngFor`? Absolutely. However, we would now need to manually determine whether an item is an alternating item. It becomes an eyesore
to see too many calculative expressions in markup, especially if they're repetitive. We'd have to do this for all the lists on the same component that we wish to
make visual distinctions for their odd-index items.

One possible fix to implement as an exercise is to let the implentor specify a `noItemsTemplate`. Right now, we would get no display, thus, worse, no visual
feedback explicitly stating that there are *really* no items in the list provided to us. We would need to also provide `defaultNoItemsTemplate` and display this
template instead of the items if there were none to display. It would be a maintenance nightmare if we had to manually create a `noItemsTemplate` template
for every single list in our app.

Another feature that we could add is the ability for users to select a single item from the incoming list, visually indicate that an item is the selected item,
and communicate back to the hosting component *that* an item was selected and *which* item was selected. This would involve exposing a `select()` function to
the `itemTemplate`, as well as whether the item is the selected ite or not so the impementor can use that value, if true, apply a CSS class that indicates
that the item was selected. Now, this feature would be a huge pain to implent over and over again for each list where we want the user to select an item.
