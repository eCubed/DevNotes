# Angular Template - Layout Templates

We will still be working with our Panel component. In this article, we will be discussing how to give the implementor the power to define a
custom layout for our entire Panel component. We will no longer be stuck with our component's decision to put both heading and content templates
inside a single div and its firm CSS decision to lay them out either horizontally or vertically.

## Layout Template Usage

We would like to be able to specify a `#layoutTemplate` that the implementor can specify the markup for our whole component in the following manner:

```html
<ec-panel heading="My Custom Heading">
  <ng-template #layoutTemplate>
    <div class="custom-panel">      
      <ec-template-placeholder [id]="headingPlaceholder"></ec-template-placeholder>      
      <ec-template-placeholder [id]="contentPlaceholder"></ec-template-placeholder>
    </div>
  </ng-template>
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

Notice that we now have the `#layoutTemplate` `<ng-template>`. Inside it is custom html markup that *includes placeholders* for the heading and content
templates. We will need to create a component for the purpose of marking where certain content would eventually be placed. We will address this
shortly.

## New Markup for Component with LayoutTemplate

First, we will need to modify the html of our Panel component to include the `#layoutTemplate` as well as a `#defaultLayoutTemplate`.

```html
<ng-container *ngTemplateOutlet="layoutTemplate || defaultLayoutTemplate">
</ng-container>

<ng-template #defaultLayoutTemplate>
  <div class="panel">
    <ng-container
      *ngTemplateOutlet="headingTemplate || defaultHeadingTemplate; context: createHeadingTemplateContext() ">
    </ng-container>
    <ng-container *ngIf="contentTemplate || defaultContentTemplate">
      <div *ngIf="isExpanded">
        <ng-container *ngTemplateOutlet="contentTemplate || defaultContentTemplate">
        </ng-container>
      </div>
    </ng-container>
  </div>
</ng-template>

<ng-template #defaultHeadingTemplate let-heading="headingText" let-expandedState="expanded" let-showOrHide="toggleShowOrHide">
  <h3 (click)="showOrHide()">{{ heading }}</h3>
</ng-template>
<ng-template #defaultContentTemplate>
  Please specify your own content for this panel!
</ng-template>
```

Let's take care of the easy case first - when the implementor doesn't specify a `#layoutTemplate`. If there was no `#layoutTeplate`, then we would
automatically apply the `#defaultLayoutTemplate` which contains the original markup we have. We just surrounded our original markup (not including
the default templates, of course), with `<ng-template #defaultLayoutTemplate>`.

As with the templates we've worked with, we also need to declare `@ContentChild` for the `#layoutTemplate` and a `@ViewChild` for the `#defaultLayoutTeplate`
in our code:

```javascript
export class PanelComponent implements OnInit {

  @Input() heading: string;

  @ContentChild('layoutTemplate', { read: TemplateRef }) layoutTemplate: TemplateRef<any>;
  @ViewChild('defaultLayoutTemplate', { read: TemplateRef }) defaultLayoutTemplate: TemplateRef<any>;

  @ContentChild('headingTemplate', { read: TemplateRef }) headingTemplate: TemplateRef<any>;
  @ViewChild('defaultHeadingTemplate', { read: TemplateRef }) defaultHeadingTemplate: TemplateRef<any>;

  @ContentChild('contentTemplate', { read: TemplateRef }) contentTemplate: TemplateRef<any>;
  @ViewChild('defaultContentTemplate', { read: TemplateRef }) defaultContentTemplate: TemplateRef<any>;

```

If we ran our app right now where we don't specify `#layoutTemplate`, then the `#defaultLayoutTemplate` will be displayed as before. If we specified
`#layoutTemplate`, though, as we proposed, our entire panel component will display *nothing*. To fix that, we will need to do quite a bit of work.


## The Template Placeholder Component

Let's review again how we would want to specify a custom `#layoutTemplate` at the front:

```html
<ec-panel heading="My Custom Heading">
  <ng-template #layoutTemplate>
    <div class="custom-panel">      
      <ec-template-placeholder id="headingPlaceholder"></ec-template-placeholder>      
      <ec-template-placeholder id="contentPlaceholder"></ec-template-placeholder>
    </div>
  </ng-template>
  ...
</ec-panel>
```

As mentioned, the `TemplatePlaceholderComponent` is used by the `#layoutTemplate` to mark where the contents of the appropriate `<ng-template>`s will
finally end up. We now present the code for this placeholder component:

```javasrcipt
export class TemplatePlaceholderComponent implements OnInit {

  @Input() id: string;

  constructor(
    private viewContainerRef: ViewContainerRef
  ) {
  }

  ngOnInit() {
  }

  clear() {
    this.viewContainerRef.clear();
  }

  append(templateRef: TemplateRef<unknown>, context?: unknown, index?: number) {
    this.viewContainerRef.createEmbeddedView(templateRef, context, index);
  }

}
```

It turns out that this template placeholder has an even more important job - to actually receive the appropriate template via the `TemplateRef<unknown>`
paramenter, along with the `context`, and `index` if applicable, instantiate template (which becomes an embedded view), and place the resulting DOM
"in its place" in the `#layoutTemplate` - done with the `append()` function. So yes, we would later call upon these placeholder components and call
the `clear()` and `append()` functions from our component's code at the right time. We address when later.

We have injected the `ViewContainerRef` service into this component's structure. `ViewContainerRef` makes the component to be able to be dynamic component
where we could load components and templates "into it" programmatically. The `createEmbeddedView` function of `ViewContainerRef` allows us to instantiate
a template, which may include the context and index (if its data were an array item). Note that we put double quotes around "into it", as in "to be able to be
a dynamic component where we could load components and templates 'into it'". `ViewContainerRef` doesn't really put components "in" existing components, but
rather as an immediate sibling of its resulting markup in html.

It is also very important that the template placeholder's markup is empty!

# Panel Component Modifications for #layoutTemplate

We now need to address the case where the user supplies a custom `#layoutTemplate` with custom html markup and particularly, the template placeholders
for placing heading and content templates. The `#layoutTemplate` will HAVE TO specify two placeholders, `headingPlaceholder` and `contentPlaceholder`.
Once that custom `#layoutTemplate` makes its way into our component via `*ngTemplateOutlet`, we will want to hunt down those placeholders so we can
fill them with the other non-layout templates. This process is quite involved, so we will break it down step-by-step.

We will now need to make changes to our component code to recognize and work with the `#layoutTemplate`:

```javascript
export class PanelComponent implements OnInit {

  @Input() heading: string;

  @ContentChild('layoutTemplate', { read: TemplateRef }) layoutTemplate: TemplateRef<any>;
  @ViewChild('defaultLayoutTemplate', { read: TemplateRef }) defaultLayoutTemplate: TemplateRef<any>;

  /* Heading and content template declarations not shown */

  @ContentChildren(TemplatePlaceholderComponent) templatePlaceholdersQueryList: QueryList<TemplatePlaceholderComponent>;
  
  headingPlaceholder: TemplatePlaceholderComponent;
  contentPlaceholder: TemplatePlaceholderComponent;
  ...
}
```

Let us first discuss how to hunt down for specific components that may be included in the custom layout template provided to us up front.

We now have `@ContentChildren(TemplatePlaceholderComponent) templatePlaceholdersQueryList: QueryList<TemplatePlaceholderComponent>;`
declaration which "flags" our Panel component to be queryable for certain elements or components. We choose `@ContentChildren` to indicate that the component 
we would be looking for is originally from the contents of an `<ng-template>`, that would finally become a part of this Panel component. Now that the content 
of `<ng-templates>` would finally make it into our component, it is now safe to assume that we can query our component's own component tree for any descendant
components of a specific component type, hence `QueryList<ComponentType>`. Our component may have lots of elements and even components in it, and we can think of
the generic parameter of `QueryList` as a top-level filter that would automatically get us *only* the components of the specified `ComponentType`. In our case, 
we want to get a hold of the two placeholders - the `headingPlaceholder` and `contentPlaceholder`, which are both of the type `TemplatePlaceholderComponent`. 
Once found, we would then dynamically load the templates via their `append(..)` function.

We also keep references to both heading and content placeholders since we will be required to find/load them in one function and then use them in another function.

### The ngDoCheck() Lifecycle Event

The `ngDoCheck()` lifecycle event occurs one time after `ngOnInit()`, and for every time Angular detects value changes made on any `@Input()`-decorated property
on the component. We have a `heading` string that *could* change over time, and for each time there is a change to `heading`'s value, `ngOnInit()` would execute.

```
  ngOnInit() {
    setTimeout(() => { this.ngDoCheck(); }, 0);
  }

  ngDoCheck() {
    this.loadTemplates();
  }

  loadTemplates() {
    if (this.headingPlaceholder != null) {
      this.headingPlaceholder.clear();
      this.headingPlaceholder.append(this.headingTemplate || this.defaultHeadingTemplate, this.createHeadingTemplateContext());
    }

    if (this.contentPlaceholder != null) {
      this.contentPlaceholder.clear();
      this.contentPlaceholder.append(this.contentTemplate || this.defaultContentTemplate);
    }
  }
```

We created a separate `loadTemplates()` function because its code block will be called from elsewhere besides in `ngDoCheck()`. Note that `ngDoCheck()` will
be called even when those placeholders haven't been found yet, since it occurs early in the component's lifecycle before our component receives the contents from
`<ng-template>`s. This is why we "null-guard" our placeholders.

For some unknown reason, there are times where `ngDoCheck()` will not automatically fire when our component is first created. This is why we explicitly call upon
it from inside `ngOnInit()`, but, for another unknown reason, we'll have to call it from `setTimeout()` with a 0-ms delay. Since `ngDoCheck()` will be called
multiple times, we will need to clear the placeholders first, then re-append the templates to reflect the change on the UI.

Note that for either template, we use the same "fallback" `||` operator to apply the default template if the expected template wasn't specified, as we have done in
the markup. Also note that we called upon the `createHeadingTemplateContext()` function that we also called upon from the markup. This way we wouldn't need to
maintain two potentially large JS objects that *have to be identical*, which would be a maintenance nightmare.

### The ngAfterContentInit() Lifecycle Event

The `ngAfterContentInit()` lifecycle event happens right after our component has received the contents of any `<ng-template>`, but before they're rendered
on the screen. For our purposes, our `templatePlaceholdersQueryList` `QueryList<ComponentType>` becomes available at this stage, *however*, we still will need
to wait for the contents of the `<ng-template #layoutTemplate>` to be accessible:

```javascript
  ngAfterContentInit() {
    this.templatePlaceholdersQueryList.changes.subscribe(c => {
      this.headingPlaceholder = this.templatePlaceholdersQueryList.find(i => i.id === 'headingPlaceholder');
      this.contentPlaceholder = this.templatePlaceholdersQueryList.find(i => i.id === 'contentPlaceholder');
      this.loadTemplates();
    });
  }
```

`QueryList<ComponentType>` has a `changes` property `Observable` that we can subscribe to once it finds all the components of type `ComponentType` in
our entire component, *including* the ones *outside* the `<ng-container *ngTemplateOutlet="layoutComponent">`. This is why we created an `@Input() id`
on the `TemplatePlaceholderComponent` so we can identify it in the `<ng-template>` upfront, and find it by id at this stage. This is where we load
the `headingPlaceholder` and `contentPlaceholder` references, and finally call `this.loadTemplates()` again. This time around, those placeholders are
no longer null provided that we didn't misspell the expected placeholder names!

If we run the application now, our panel will display as expected, according to the provided custom `#layoutTemplate`.


## Next Steps

It was quite a feat to use Angular's templating features and some of its lifecycle event hooks to be able to pull layout templates! It was all worth it because
we can now at will "draw" any Panel component and use it to maintain expanded/contracted state per instance.

There is one thing to take care of. We didn't get a hold of the `QueryList<ComponentType>`'s `change` subscription so we can unsubscribe from it. We can
resolve that later as it is a topic aside from layout templating.

Now with all this templating knowledge, we could start creating other components that could use use templating - components that would encapsulate some unique
functionality but would let the implentor render it entirely.





