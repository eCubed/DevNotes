# Angular Templating - Exposing Data and Functionality to Templates

We left off last article with discussing multiple templates that a component might expect. We are still working with the Panel component. At this time, it
expects only one template, and that is the `#contentTemplate`. Suppose that we want the user to be able to customize the heading as well.
Right now, we are stuck with an `<h3>` element that outputs the heading value passed into our component, and clicking on it toggles
the `isExpanded` state. What if the implementor wants to totally change the heading's markup and look overall, and still want to
display the heading text as passed in, and be able to click on something to toggle the components `isExpanded` state? We certainly
can do those things, but for now, let's start with adding the `#headingTemplate`.

# Adding a New Template

When we plop a Panel component in markup, we want to be able to specify something like the following:

```html
<ec-panel heading="My Custom Heading">
  <ng-template #headingTemplate>
    <div class="heading">
      <span class="heading-text">My Custom Heading</span>
      <span class="spacer"></span>
      <span class="toggle-expand">X</span>
    </div>
  </ng-template>

  <ng-template #contentTemplate>
    Custom content
  </ng-template>
</ec-panel>
```

It seems like the `#headingTemplate` is meant to display the heading text in a toolbar and it won't be clickable for toggling. There
would be a small x-button on the right-hand side of the toolbar that is meant to be clicked for toggling the `isExpanded` state.

Since we introduced a new template, the `#headingTemplate`, we will need to make a few modifications to our component. First, we
add a `@ContentChild` for the template the same way we have a `@ContentChild` for the `#contentTemplate`:

```javascript
export class PanelComponent implements OnInit, DoCheck, AfterContentInit, OnDestroy {

  @Input() heading: string;

  @ContentChild('headingTemplate', { read: TemplateRef }) headingTemplate: TemplateRef<any>;
  @ContentChild('contentTemplate', { read: TemplateRef }) contentTemplate: TemplateRef<any>;
  isExpanded = true;
  ...
}
```

Then, in our component's markup, we replace that `<h3>` tag with an `<ng-container>` placeholder for the `#headingTemplate`. This way, the contents
of `<ng-template #headingTemplate>` will be put in place where `<ng-container *ngTemplateOutlet="headingTemplate">` is declared:

```html
  <div class="panel">
    <ng-container
      *ngTemplateOutlet="headingTemplate">
    </ng-container>
    <div *ngIf="isExpanded">
      <ng-container *ngTemplateOutlet="contentTemplate">
      </ng-container>
    </div>
  </div>
```

Now, the implementor's heading markup will show instead of that `<h3>` tag we were originally stuck with.

So far, we can't grab, thus we can't display the heading text passed into our component, nor can we toggle the `isExpanded` state
from the custom `#headingTemplate`. We will address this shortly.

# Exposing Values to a Custom Template
There are times that an `<ng-template>` up-front (that is, when plopping the markup of our component onto a host component) will need data from inside our
component so that the implementor can display their values wherever in that custom `<ng-template>`, or use those values for visual effects, such as binding
them to CSS values. Currently, our `<ng-template #headingTemplate>` receives nothing from inside our Panel component. We need to give our custom `<ng-template>`
the heading text and the `isExpanded` state. This act is referred to as **exposing values to a template**, and those set of values is known as the **template
context**.

The **template context** *is* a regular JS object that we can specify in our component's html:

```html
  <div class="panel">
    <ng-container
      *ngTemplateOutlet="headingTemplate; context: { headingText: heading, expanded: isExpanded }">
    </ng-container>
    <div *ngIf="isExpanded">
      <ng-container *ngTemplateOutlet="contentTemplate">
      </ng-container>
    </div>
  </div>
```

Let's focus on `<ng-container *ngTemplateOutlet="headingTemplate; context: { headingText: heading, expanded: isExpanded }">`. The first "parameter"
of `*ngTemplateOutlet`, as we have seen before, is the identifier of the template whose contents we want to display it its (the `<ng-container>`'s) place.
The second "parameter" is the template context object. It is important to know that we can put *any* property names we want (as long as they follow JS variable
naming rules), and *however many* properties we need. Each of those properties will be accessible to the front `<ng-template>` as we'll see shortly.

In our case, our template context is a JS object with the properties `headingText`, which we'll assign the value of the component's `heading` @Input property,
and `expanded` to be the component's `isExpanded` property's value. Now, let's see how we can access the context in the front template:

```html
<ec-panel heading="My Custom Heading">
  <ng-template #headingTemplate let-heading="headingText" let-expandedState="expanded">
    <div class="heading">
      <span class="heading-text">{{ heading }}</span>
      <span class="spacer"></span>
      <span class="toggle-expand">{{ expandedState ? v : ^ }}</span>
    </div>
  </ng-template>

  <ng-template #contentTemplate>
    Custom content
  </ng-template>
</ec-panel>
```

`<ng-template>` lets us leverage the `let-xyz="contextPropertyName"` attribute. The `let-` portion is always going to be `let-`. The xyz is going to be the
variable name we'll use within our `<ng-template>`, which would be like an *alias* to `contextPropertyName` which is a property by name from the context 
template we've declared from `*ngTemplateOutlet`.

In our case, for our `<ng-template>`, `let-heading="headingText"` means we'll want to use the local variable `heading` to refer to the `headingText`
property we've declared from `*ngTemplateOutlet*. We finally use `heading` to display its values with the familiar double-curly-braces expression 
`{{ heading }}` inside the heading `<span>`.

Similarly, we are exposing the context's `expanded` property *as* `expandedState`, whose value we'll use to determine whether to display the character `v`
or `^`.

If we run the application now, we will see the customized heading *and also* the contents of the `#contentTemplate` because `isExpanded` *was* initialized
to true by default.

We're getting somewhere, but now, we still can't click on anything in the custom `#headingTemplate` to toggle the visibility of the `#contentTemplate`. We will
address how to let the front access a function from inside our component.

## Exposing Functions to Custom Templates

We know so far that we have written the function `toggleExpanded`:

```javascript
toggleExpanded() {
  this.isExpanded = !this.isExpanded;
}
```

If we can expose any property we want to the template context, we may guess that it *is* also possible to expose a function the same way, and that guess *is*
correct! Let's go ahead and add another property to the context in the `*ngTemplateOutlet`:

```html
  <div class="panel">
    <ng-container
      *ngTemplateOutlet="headingTemplate; context: { headingText: heading, expanded: isExpanded, toggleShowOrHide: toggleExpanded }">
    </ng-container>
    <div *ngIf="isExpanded">
      <ng-container *ngTemplateOutlet="contentTemplate">
      </ng-container>
    </div>
  </div>
```

Note that we're exposing the `toggleExpanded()` *as a property*, which is why it's just `toggleShowOrHide: toggleExpanded`, not `toggleShowOrHide: 
toggleExpanded()`, which is wrong because `toggleExpanded` returns `void`, and we certainly don't want to set `toggleShowOrHide` to `void`. Now,
how would we receive the function in the front `<ng-template>`?

```html
<ec-panel heading="My Custom Heading">
  <ng-template #headingTemplate let-heading="headingText" let-expandedState="expanded" let-showOrHide="toggleShowOrHide">
    <div class="heading">
      <span class="heading-text">{{ heading }}</span>
      <span class="spacer"></span>
      <span class="toggle-expand" (click)="showOrHide()">{{ expandedState ? v : ^ }}</span>
    </div>
  </ng-template>

  <ng-template #contentTemplate>
    Custom content
  </ng-template>
</ec-panel>
```

To expose a function to the front template is done in a similar fashion as with exposed variables. We exposed the `toggleExpanded()` function as 
`toggleShowOrHide` in the template context, and we're going to use the context's `toggleShowOrHide` in the front template simply as `showOrHide`. We then
finally hook it up to the `(click)` event of that `toggle-expand` `<span>` element with `(click)="showOrHide()"`.

If we run the app and try to click over and over again, we will notice that expanding and contracting *did not visibly happen*. It seems like the component's
`isExpanded` state remains true no matter what.

If we go back to `toggleExpanded()` and see what's happening with a `console.log()`, we will see that the value of `this.isExpanded` *does* toggle between
true and false when clicked! *What* is going on? Why is the toggling values of `this.isExpanded` not reflected in the UI?

It all has to do with the context of `this`. It is easy to believe, especially that if we have back-end OOP backgrounds, that the context of `this` of a
function belonging to a class, will *always* be the instantiated object, even if we pass that function around our app. Though our component is truly a class
in TypeScript, the context of `this` inside the class's functions *can change* to *no longer be* our component. We lose the context of `this` as our 
component when we pass it around as callbacks or set it equal to a property of another class or object. We created the property `toggleShowOrHide` *in* our
*template context*, and set it to point to our component's original `toggleExpanded`. Now, though `toggleShowOrHide()` really points to our component's
original `toggleExpanded`, the context of `this` when when we call `toggleShowOrHide()` is now the template context, and no longer the original component
as we expect.

To "cure" this, we will need to rewrite our function `toggleExpanded()` *as a property* that is set equal to `*an arrow function*`:

```javascript
toggleExpanded = () => {
  this.isExpanded = !this.isExpanded;
```

When we change from a pure function to a property set to an arrow function, that ensures that the context of *this* will *always* be the component where the
arrow function was originally declared in, no matter how far we pass our function throughout the app.

If we run the application again, we will see that clicking on the `toggle-expand` `<span>` will *finally* physically show and hide the content!

## Next Steps

What if the implementor of our panel doesn't specify the `#headingTemplate` or the `#contentTemplate`? The easy answer to that is that the template they omit
will just not show up on the screen. Should we, at our component's `ngOnInit()` detect that if an expected template doesn't exist, we should thrown an error? 
Or, do we supply a "backup" template and show that instead if they omit it up front? We will tackle these issues as we work with what we'll call "default templates".

