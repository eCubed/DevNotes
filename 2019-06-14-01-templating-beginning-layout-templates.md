# Angular Templating - Beginning Layout Templates

In this article, we will revisit the panel component, introduce some problems with it, and what we'll need to do *to start* making fixes. We only
say *to start making fixes* because there will be a lot of Angular concepts and a few hacks necessary to pull the ultimate feat - let the implementor
*provide* the *entire layout* for our component, not just the layout of each individual item via `#itemTemplate`. Before we get into modifying
the panel component for further robustness, we will first review the existing code:

```html
<div class="panel">
  <h3 (click)="toggleExpanded()">{{ heading }}</h3>
  <div *ngIf="isExpanded">
    <ng-container *ngTemplateOutlet="contentTemplate">
    </ng-container>
  </div>
</div>
```

```javascript
...
export class PanelComponent implements OnInit {
  @Input() heading: string;
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

And its usage on a host component:

```html
<ec-panel heading="My Panel Heading">
  <ng-template #contentTemplate>
    Custom content
  </ng-template>
</ec-panel>
```

## Allow Heading Customization

The above `PanelComponent` is already robust with letting us put any content we want via the `#contentTemplate` `<ng-template>`. It's also
a nice touch to add the capability to hide and show the content when the heading is clicked. There is a problem with this - we are stuck with the
`<h3>` tag for the heading of the panel. Furthermore, since we're stuck with that tag, we are also stuck with whatever CSS we'll apply to that tag
from inside our component! This means that our panel will *always* look the same *no matter* which app we'll put it on. There is no straight-forward
way, and might we go ahead and say that it is virtually impossible to attempt to alter the CSS rules of a component from outside it. No, our universal
CSS rules that target even that `<h3>` element will not apply to the CSS of components, because CSS for components *is isolated* by Angular's decision!

In order to combat this look issue, we will instead let the implementor specify a `#headerTemplate` so they can put any markup they want and apply
any CSS rules they want for the header's custom markup. In the end, the component *will accept everything* provided inside `<ng-template>`s it
accepts, *including all CSS rules* decided by the hosting component or the whole app!

First, let us go ahead and put `<ng-template #headingTemplate>` in the usage of our component:

```
```html
<ec-panel heading="My Panel Heading">
  <ng-template #headingTemplate let-heading>
    <div class="heading">
      <span class="heading-text">{{ heading }}</span>
      <span class="spacer"></span>
      <span class="close">X</span>
    </div>
  </ng-template>
  <ng-template #contentTemplate>
    Custom content
  </ng-template>
</ec-panel>
```

Instead of the simple and "un-restylable" `<h3>` tag inside our component itself, we let our user decide the markup and custom CSS that our component
will just accept and display as expected. Now, we will want our component to expose the `heading` variable so we can display it. We also provide the
CSS (scss) so we could see the effect of letting the implementor have total control of the appearance of an `<ng-template>` recognized by our component:

```css
.heading {
  display: flex;
  flex-direction: row;

  color: white;
  background-color: #008000;

  & > * {
    box-sizing: border-box;
    padding: 10px;
  }

  .heading-text {
    flex: 1 0 10px;
  }

  .spacer {
    flex: 1;
  }

  .close {
    flex: 0 0 30px;
    background-color: #ccffcc;
    color: #008800;
    font-weight: bold;
    cursor: pointer;
  }
}
```

## Exposing Functions to the Template

We notice that we [temporarily] lost the ability to expand and collapse the content from inside our panel. Since we now create the custom heading markup,
we will now need some way to hook to the `toggleExpanded()` function that is inside our component's code, from the `<ng-template>` up front. Since
the `context` object parameter of `*ngTemplateOutlet` allows us to expose values from inside our component, to an `<ng-template>` up front, we
*can also expose functions*! We add the `toggleExpanded` function to the `<ng-container>` of the `headingTemplate`:

```html
<div class="panel">
  <ng-container *ngTemplateOutlet="headingTemplate; context: { $implicit: heading, toggleShowOrHide: toggleExpanded } ">
  </ng-container>
  <div *ngIf="isExpanded">
    Is expanded? {{ isExpanded }}
    <ng-container *ngTemplateOutlet="contentTemplate">
    </ng-container>
  </div>
</div>
```

Note that we expose the `toggleExpanded()` function as `toggleShowOrHide`. We must remember that when we use functions as property values, we do not
include the parentheses, since including the parentheses means to actually execute the function, which we're not looking to do at this stage.

Now, let's receive this exposed function in the `<ng-template #headingTemplate>`:

```html
<ec-panel heading="My Custom Heading">
  <ng-template #headingTemplate let-heading let-showOrHide="toggleShowOrHide">
    <div class="heading">
      <span class="heading-text">{{ heading }}</span>
      <span class="spacer"></span>
      <span class="close" (click)="showOrHide()">X</span>
    </div>
  </ng-template>
  <ng-template #contentTemplate>
    Custom content
  </ng-template>
</ec-panel>
```

Notice that we can also "let-rename-as" the function `toggleShowOrHide` as exposed by the `context` the same way with actual variable values. Now, we
hook up that function to the `(click)` event of that close button. This time, we include the parentheses as we're intending to execute the function. If
we run the application, we will NOT see the content appear/disappear. But indeed, the component's `toggleExpanded()` function *was* actually called, and
we can examine with a `console.log()` inside that function that the `isExpanded` value toggles between subsequent clicks. Alas, that change isn't reflected
in the UI. The cure for that is to rewrite `toggleExpanded()` as a property of the same name set equal to an anonymous/arrow function:

```
toggleExpanded = () => {
  console.log(`toggleExpanded Shown!`);
  this.isExpanded = !this.isExpanded;
}
```

Why? When a function is passed around and called from outside the component that defines it, the function's `this` is *no longer* the component itself. But
when passed around as a property *that is* an arrow function, the `this` will retain its original source or context, the component where it's defined!

If we run the app again, we will see the content show and hide as we click through it.

## Next Steps

We have just given the app total control of the panel's heading - both markup and CSS so we're no longer stuck with a single `<h3>` element to render it as it
was when our component forced it. We can go a step further to give the app *even more control* of the markup *of the whole component*. Right now in our component,
we are *stuck* with that top-level `<div class="panel">` and we are also *stuck* with whatever CSS we put in our component's CSS or SCSS files no matter where
we take our component.

Suppose we want to use this same component on one page where heading and content should be displayed sideways, but on another page, panels would be displayed
with the heading on the top as expected - and we want to use the same panel component. We couldn't pull that with the panel component we have now because we're
stuck with the `<div>` and its CSS that forces the header to *always* be above the content.

In the next article, we'll explore how we can create layout templates so we will gain *full customizable control* of our component while maintaining the show/hide
functionality no matter what markup the implementor decides.