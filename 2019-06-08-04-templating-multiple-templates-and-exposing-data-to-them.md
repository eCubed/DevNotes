# Angular Templating - Multiple Templates and Exposing Data To Them

We left off last article with discussing multiple templates that a component might expect. We are still working with the Panel component. At this time, it
expects only one template, and that is the `#contentTemplate`. Suppose that we want the user to be able to customize the heading.
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

Then, in our component's markup, we replace that `<h3>` tag with an `<ng-container>` placeholder for the `#headingTemplate`:

```html
  <div class="panel">
    <ng-container
      *ngTemplateOutlet="headingTemplate">
    </ng-container>
    <ng-container *ngIf="contentTemplate">
      <div *ngIf="isExpanded">
        <ng-container *ngTemplateOutlet="contentTemplate">
        </ng-container>
      </div>
    </ng-container>
  </div>
```

This way, the implementor's heading markup will show instead of that `<h3>` tag we were originally stuck with.

So far, we can't grab, thus we can't display the heading text passed into our component, nor can we toggle the `isExpanded` state
from the custom `#headingTemplate`. We will address this shortly.

# Exposing Values to a Custom Template


