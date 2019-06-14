# Angular Templating - Accessing External Templates From Components

Suppose that we want our app or website to display lots of content in panels. We would explicitly create the following markup:

```html
<div class="panel">
  <h3>Heading Text</h3>
  <div>
    Any content we want.
  </div>
</div>
```

However, we'll need to create this markup all over our app for any content we want to put in a panel with a heading. At first, writing this markup does not
seem to be a bother even if we'll do it many times across many different components. What if we wanted to make that panel be able to collapse and expand upon
clicking on the heading? Then we'd have to modify our markup:

```
<div class="panel">
  <h3 (click)="toggleExpand()">Heading Text</h3>
  <div *ngIf="isExpanded">
    Any content we want.
  </div>
</div>
```

Then we would have to keep track of the expanded state in our hosting component and create the `toggleExpand()` function:

```js
...

isExpanded: boolean = true;

toggleExpand() {
  this.isExpanded = !this.isExpanded;
}
```

We can already see that we will have to create the expanded state and the toggle function on *every* piece of content we want to put in a panel. What if we have
more panels in one component and we'd like to individually and *independently* maintain their expanded state? We would then need to create an expanded state
for each one, and a toggle function for each one. This is a huge problem.

Then we might think that we may want to create separate components for every single one of our panels. While that will solve the problem of keeping the expanded
state and the toggle function isolated from other similar components, that is still very tedious. Yes, there is a better way - a way where we will create
one and only one panel component that will maintain and manage expanded state *automatically* for every single instance we would plop down the component,
even multiple of them in one component!

So, we would be able to finally plop the following component in markup:

```html
<lib-panel heading="Panel Title">
 ...
 The content we want to show.
 ...
</lib-panel>
```

And no, we don't have to keep expanded state explicitly as we have done before. However, notice that we have put ... around "The content we want to show."
We will need to do something there, and we will address that shortly.

## Using ng-template to Specify a Template Inside Custom Component Tags

If we recall from the previous section, we can use `<ng-template>`s to specify some piece of content that won't initially show up until the conditions
are right. Indeed, we can also put an `<ng-template>` *inside* the markup of our custom component:

```html
<lib-panel heading="Panel Title">
  <ng-template #contentTemplate>
    The content we want to show.
  </ng-template>
</lib-panel>
```

Now, let's pretend for now that we have already created the actual component for the panel, and we may even have included an @Input so we can have an attribute
for the panel's title. Maybe to our surprise, we did not get an error when we ran the app. And since `<ng-template>` isn't meant to show up right away until
the right conditions, we did not see the text "The content we want to show" on the page.

## Making our Component Recognize ng-templates

There are a few things that we'll need to do in our custom components to be able to recognize ng-templates and finally display those ng-templates' contents.
But first, notice that in the markup above, we *identified* the ng-template with an #identifier. We need this identifier so that our component can get a hold
of it for manipulation.

The first thing we'll need to do is to declare a @ContentChild for that ng-template, in the .ts file:

```js
export class PanelComponent implements OnInit {

  @Input() heading: string;
  @ContentChild('contentTemplate', {read: TemplateRef}) contentTemplate: TemplateRef<any>;

  constructor() { }

  ngOnInit() {
  }

}

```

That certain looks very verbose, but let's break it down:

* The @ContentChild decorator indicates that our component would be looking for certain elements or components that may have been put between its tags on 
usage of our component.
* The first parameter of @ContentChild *is* the identifier. In our case, it is the identifier `contentTemplate` of our `<ng-template>`
* The second parameter is an object *always* with the property name `read`, with the value which is a class name. Since we want to recognize an `<ng-template>`,
we want to specify `TemplateRef` for the value of `read`. It is possible to put other classes besides `TemplateRef`, but that's beyond the scope of this article.
* We then of course, create the property of our component class, as we normally do. We name it *exactly* after the identifier by convention and for consistency. The
data type should be `TemplateRef<any>` if we're expecting to work with an `<ng-template>`.

## Placing the Contents of <ng-template> in the Component's Markup

Now that we declared a @ContentChild in our .ts file for our component to recognize a template by an identifier, we may ask where in our component's markup do we
stick the contents of the `<ng-template>`, but more importantly, *how*? Let's tackle the *where* first since it's the easier one - we can put the contents of
the `<ng-template>` *anywhere.* In our case, we would want them not in the `<h3>` (of course, that's for the heading), but inside the inner div.

Now that we decided where to put the contents of the `<ng-template>`, there's got to be some placeholder where we'll place them:

```html
<div class="panel">
  <h3>{{ heading }}</h3>
  <div>
    <ng-container *ngTemplateOutlet="contentTemplate">
    </ng-container>
  </div>
</div>
```

We use `<ng-container>` as the *placeholder* for the contents of our `<ng-template>`. Now, in order to pick out which `<ng-template>`'s contents should
be put in place of the `<ng-container>`, we specify the template identifier with the structural directive *ngTemplateOutlet*, yes, with the leading asterisk.

If we run the application now, we will see the contents of our `<ng-template>` as we've marked up.

If we go ahead and plop more instances of our panel component `<lib-panel>` with custom template component, we will see each one.

## Finishing Up

Now, let's add the expanded state and toggle functions to our custom component, and also make modifications in the markup to show the effect of expanding
and contracting:

```js
export class PanelComponent implements OnInit {

  @ContentChild('contentTemplate', {read: TemplateRef}) contentTemplate: TemplateRef<any>;

  isExpanded = true;

  constructor() { }

  ngOnInit() {
  }

  toggleExpanded() {
    this.isExpanded = !this.isExpanded;
  }
}

```

```html
<div class="panel">
  <h3 (click)="toggleExpanded()">Heading</h3>
  <div *ngIf="isExpanded">
    <ng-container *ngTemplateOutlet="contentTemplate">
    </ng-container>
  </div>
</div>
```

If we run our app now, each and every single instance of the panel component will show its custom content, and will manage and maintain its own expanded state
*independently* from the other panel instances. Best of all, we no longer have to explicitly maintain expanded state on every single instance on every single
host component!

## Next Steps

One question that arises is whether it is possible for a component to recognize more than one `<ng-template>`, and the answer to that is yes. Maybe it doesn't
really apply to a simple panel component where we merely and always want to display the contents of the `<ng-template>`, but there are instances where we might
want to show contents of one `<ng-template>` vs. another `<ng-template>`'s.

Also, it is possible to let `<ng-templates>` we put inside our component's tags, to get hold of certain values passed into our component or generated from inside our
component for some additional control of how to render contents. We will explore multiple `<ng-template>`s and exposing some values to `<ng-template>`


