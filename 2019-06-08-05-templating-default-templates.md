# Angular Templating - Default Templates

We are still working on our Panel component. So far, we're having the user specify the `#headingTemplate` and the `#contentTemplate`, and
even handing some internal values and functions to those custom templates for enhanced customizability. Let's review our component's markup so far:

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

We will now start thinking about what would happen if the implementor omits any of those templates up-front.

If went back to our component's usage and remove the templates, like this:

```html
<ec-panel heading="My Custom Heading">
</ec-panel>
```

We would get an error. Why? Because we would be trying to display a template that doesn't exist, let alone try to give context to a template that
doesn't exist!

So, what can we do? One possibility is to put `<ng-container *ngIf="expectedTemplate != null">` around `<ng-container *ngTemplateOutlet="expectedTemplate">`.
This way, if the template wasn't specified, we simply don't show it. We would get no visual display for our component, which isn't a very good way
to communicate to the implementor that they are may be something missing, let alone *what* may be missing. What we'll need to do is to instead
provide **default templates**, which would ultimately be shown in case an expected template may not have been supplied up front.

## Creating Default Templates

We would specify **default templates** right in our component's markup:

```html
  <div class="panel">
    <ng-container
      *ngTemplateOutlet="headingTemplate || defaultHeadingTemplate; context: { headingText: heading, expanded: isExpanded, toggleShowOrHide: toggleExpanded }">
    </ng-container>
    <div *ngIf="isExpanded">
      <ng-container *ngTemplateOutlet="contentTemplate || defaultContentTemplate">
      </ng-container>
    </div>
  </div>
  <ng-template #defaultHeadingTemplate let-heading="headingText" let-expandedState="expanded" let-showOrHide="toggleShowOrHide">
    <h3 (click)="showOrHide()">{{ heading }}</h3>
  </ng-template>
  <ng-template #defaultContentTemplate>
    Please specify your own content for this panel!
  </ng-template>
```

In our component's `<ng-container *ngTemplateOutlet...>`, we would specify the defaultTemplate *as a fallback* to our expected template using
the `||` operator. For example, `<ng-container *ngTemplateOutlet="headingTemplate || defaultHeadingTemplate;...">` means "apply the
`#headingTemplate` if it exists, but if it doesn't, apply the `#defaultHeadingTemplate` which will surely exist because we'll specify it
right here in our markup."

Notice that the declaration of the default templates look *very similar* to the templates we would specify up front when we plop our Panel
component onto a host component's markup. It's just any `<ng-template>` that holds what would be displayed whenever needed, and it might take in
values and functions from `let-xyz="contextPropertyName"` attributes from our component's internals.

We are not done yet. Now, back in our component's TS file, we have to reference those default templates although we won't access the directly
from code just yet:

```js
export class PanelComponent implements OnInit {

  @Input() heading: string;

  @ContentChild('headingTemplate', { read: TemplateRef }) headingTemplate: TemplateRef<any>;
  @ViewChild('defaultHeadingTemplate', { read: TemplateRef }) defaultHeadingTemplate: TemplateRef<any>;

  @ContentChild('contentTemplate', { read: TemplateRef }) contentTemplate: TemplateRef<any>;
  @ViewChild('defaultContentTemplate', { read: TemplateRef }) defaultContentTemplate: TemplateRef<any>;

  isExpanded = true;
  ...
}
```

We declare a `@ViewChild` property for our default templates. Notice that the declaration is very similar to that of our expected templates
with one key difference. Our expected templates are marked with `@ContentChild` which, as a review, tells our component to go look for the
expected template up front - where the template *literally* is a *child* of our component's markup tag that contains *the content*. `@ViewChild`,
on the other hand tells our component to go look for the template *in our component's markup*.

If we didn't declare a `@ViewChild` property for our default templates, we wouldn't be able to pass it to `*ngTemplateOutlet` since its first
"parameter" *is* required to be a property of our component. Had our default templates been declared as `@ContentChild`, our component would be
looking for our default templates up front, which they wouldn't be, so that would cause an error.

## Creating Template Context From Code Instead

If we look at the `*ngTemplateOutlet` declaration for the heading templates, it looks very long because of all of those values we would want
to expose to our templates. We'll want to reduce that quite a bit, since more complicated templated components in later development would demand
far more properties to expose to templates.

We can go ahead and create a function in code that would simply return the heading template context:

```javascript
  createHeadingTemplateContext() {
    return {
      heading: this.heading,
      toggleShowOrHide: this.toggleExpanded,
      isExpanded: this.isExpanded
    };
  }
```

This would declutter our markup. Now, it's more condensed:

```html
 <ng-container
   *ngTemplateOutlet="headingTemplate || defaultHeadingTemplate; context: createHeadingTemplateContext()">
 </ng-container>
```

Remember that exposing our `this.toggleExpanded` function would still change the context of `this` when it's called via the context's `toggleShowOrHide`
property!

There is another reason why we may want to do create the context from code instead, and that's because we would also call upon that 
`createTemplateContext` function from a different place in code other than from our component's markup.

## Default Template Considerations

When we create default templates, we are making firm decisions on what *exactly* should be rendered if the implementor doesn't provide expected
templates. We also made a firm decision not to throw errors if we detect a missing template from up front for our Panel Template. Notice that we
specified specific CSS class names in our default templates, whose CSS styles would be declared from our component's .css or .scss files and would
*always* have that look no matter what app we put it in. We could also make a decision to purposely make our default templates to look terrible
without any styling at all to encourage implementors to supply a custom template.

One particular curious case is that of our default content template. Let us remember that the point of this panel is to surround any existing
content with preset markup and a heading, which automatically means that the content template *should* probably be required that if omitted, we
should crash the application if we detect its absence. It really depends on the templated components' developers to decide whether to throw an
error or still provide a default template if a required template is missing. We stuck with the decision to provide a default template with a
message "Please specify your own content for this panel!".

## Next Steps

So far, our Panel coponent is very robust that it lets implementors define custom markup via templates, and also would show default templates
should they decide not to or forgot to specify a template. There is one niggling issue, though. We surrounded everything with a `<div>` with
the class name "panel" in our component's markup. No matter what the header markup and content end up being, they will both be placed one on
top of each other, inside that div, and we're stuck with that fix. Suppose that for some odd reason, we styled that div to lay out its children
horizontally. Then we would be stuck with that horizontal layout no matter how customized the heading and content templates would be - they would
always show up side-by-side. Likewise, if we styled that div to lay out its children vertically, we would be stuck with that vertical layout.
At this point, we have no way for the implementor up front to control the `layout` of those templates since we're stuck with the layout
decided by our template's markup. We will explore this issue in the next article.

