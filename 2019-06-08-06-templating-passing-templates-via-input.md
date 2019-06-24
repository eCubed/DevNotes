# Angular Templating - Passing Templates Via Input

Before we head out to explore the capability of giving the implementor full control of the component's layout, we'll need to explore another way
that our components receive templates. Right now, we would plop our component's markup on a host component (statically), and explicitly specify templates 
between our component's markup tags. As a review, these templates we provide between our component's markup tags are considered content children. We mark
`TemplateRef`s with the `@ContentChild` decorator to let our component know to expect statically-placed `<ng-template>` tags inside our component's
markup. 

Specifying template content children is suitable if we already know *at design time* what the exact templates will be. There will be times, though, that
we wouldn't know what templates to specify for our component's content children. What if there is some logic in the hosting component that would need to
decide which templates to "feed" to our templated component? What if our host component *itself* were dynamically instantiated, and during its instantiation,
we would specify from somewhere templates that we would need to pass to our templated component? We can certainly pass `TemplateRefs` around, however,
we *cannot* receive unknown templates via content children. They're actually just like any object in JS.

In this article, we will be continuing work on our Panel component.

# @Input of Type TemplateRef
Well just say this straight up - yes, we can pass template refs with `@Input`: `@Input('some-template') someTemplateFromInput: TemplateRef<any>;`.
By convention, we would suffix `-FromInput` because we *also* want to retain the `@ContentChild` version of our template as we already have it. So, for
our case, we should add an `@Input()` for each of the heading and content templates:

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

Notice that we used kebab-case because `@Input` flags our component to go look for values in our component's tag's attributes, whose attribute naes are in
kebab-case. Note also that we did not specify `{ read: TemplateRef }` for the `@Input`, because it doesn't take that argument.

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
