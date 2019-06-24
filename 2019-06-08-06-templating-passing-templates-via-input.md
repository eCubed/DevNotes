# Angular Templating - Passing Templates Via Input

Before we head out to explore the capability of giving the implementor full control of the component's layout, we'll need to explore another way
that templates get to our component. We are still going to work with our Panel Component. Right now, we would plop our Panel component's markup on
a host component (statically), and explicitly specify templates between our component's markup tags. These templates are considered content children.
Specifying template content children is suitable if we already know *at design time* what the exact template will be. There will be times, though,
that when we plop our templated component on a host component, we wouldn't know right then and there *exactly* what the actual templates would be
that would be used to render the content. There will come a time where if our host component where we plop our templated component's markup, would be
dynamically instantiated, and we wanted that host component to be able to take in `TemplateRefs` to pass to our templated components. So, how would
we pass unknown templates to our templated component? Certainly not through `<ng-template>` content children!

# @Input of Type TemplateRef
Well just say this straight up - yes, we can pass template refs with `@Input`: `@Input('some-template') someTemplateFromInput: TemplateRef<any>;`.
By convention, we would suffix `-FromInput` because we *also* want to retain the `@ContentChild` version of our template as we have it. So, for our
case, we should add an `@Input()` for each of the heading and content templates:

```javascript
export class PanelComponent implements OnInit, OnDestroy {

  @Input() heading: string;

  @ContentChild('headingTemplate', { read: TemplateRef }) headingTemplate: TemplateRef<any>;
  @Input('heading-template') headingTemplateFromInput: TemplateRef<any>;
  @ViewChild('defaultHeadingTemplate', { read: TemplateRef }) defaultHeadingTemplate: TemplateRef<any>;

  @ContentChild('contentTemplate', { read: TemplateRef }) contentTemplate: TemplateRef<any>;
  @Input('content-template') contentTemplateFromInput: TemplateRef<any>;
  @ViewChild('defaultContentTemplate', { read: TemplateRef }) defaultContentTemplate: TemplateRef<any>;
  ...
}
```

Notice that we used kebab-case because `@Input` means that our properties become our component's tag's attributes, which are in kebab-case. Note also
that we did not specify `{ read: TemplateRef }` for the `@Input`, because it doesn't take that argument.

# Template Resolution between @ContentChild, @Input, and Default Template
So far, our component *can* take in a template via `@Input` optionally. But right now, we don't know how our app decides which one to use if both
`@ContentChild` and `@Input` were specified. We will prefer the `@ContentChild` if it exists, and if it doesn't, we'll resort to `@Input` if it
exist. If `@Input` doesn't exist, then we fall back to the default template. This is a matter of `||`-stringing them in the order `@ContentChild || 
@Input || DefaultTemplate`. We will now need to modify our component's markup to also recognize the `@Input`:

```html
  <div class="panel">
    <ng-container
      *ngTemplateOutlet="headingTemplate || headingTemplateFromInput || defaultHeadingTemplate; context: createHeadingTemplateContext() ">
    </ng-container>
    <ng-container *ngIf="isExpanded">
      <ng-container *ngTemplateOutlet="contentTemplate || contentTemplateFromInput || defaultContentTemplate">
      </ng-container>
    </ng-container>
  </div>
  <ng-template #defaultHeadingTemplate let-toggle="toggleShowOrHide">
    <h3 (click)="toggle()">{{ heading }}</h3>
  </ng-template>
  <ng-template #defaultContentTemplate>
    You have not supplied a #contentTemplate. Please supply one to replace this message.
  </ng-template>
```

As we can see, the `||`-stringing is becoming long, which becomes an eye-sore and maintenance nightmare in our markup. We'll transfer those to
functions to clean up our html a bit:

```javascript
  resolveHeadingTemplate() {
    return this.headingTemplate || this.headingTemplateFromInput || this.defaultHeadingTemplate;
  }

  resolveContentTemplate() {
    return this.contentTemplate || this.contentTemplateFromInput || this.defaultContentTemplate;
  }
```

Now, we have a much cleaner html:

```html
  <div class="panel">
    <ng-container
      *ngTemplateOutlet="resolveHeadingTemplate(); context: createHeadingTemplateContext() ">
    </ng-container>
    <ng-container *ngIf="isExpanded">
      <ng-container *ngTemplateOutlet="resolveContentTemplate()">
      </ng-container>
    </ng-container>
  </div>
  <ng-template #defaultHeadingTemplate let-toggle="toggleShowOrHide">
    <h3 (click)="toggle()">{{ heading }}</h3>
  </ng-template>
  <ng-template #defaultContentTemplate>
    You have not supplied a #contentTemplate. Please supply one to replace this message.
  </ng-template>
```

It is also useful to create these resolution functions since they will be called from code later when we need to dynamically instantiate a template.
More on that later when we start developing custom layouts!
