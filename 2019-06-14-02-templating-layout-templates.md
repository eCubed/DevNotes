# Angular Templating - Layout Templates

In the previous article, we have discussed giving the implementor full control of the markup of our entire component, while exposing internal values and
functions to the front template and maintaining the component's core functionalities. We will still be working with the panel component. The panel component
right has `<div>` that forces the header to *always* be above the content, and we don't necessarily want our panel laid out that way, and no, we will not
create separate panel components that have layouts we like. We will stick with the one panel we have but modify it so that the user can lay it out in
whatever manner they want.

## Layout Template Usage

We would like to be able to specify a `#layoutTemplate` that the implementor can specify the markup for our whole component in the following manner:

```html
<ec-panel heading="My Custom Heading">
  <ng-template #layoutTemplate>
    <div class="panel">      
      <ec-template-placeholder [id]="headingPlaceholder"></ec-template-placeholder>
      <div>
        Whatever we want, probably ad space that we do NOT want to be in the same div as the content!!!
      </div>
      <ec-template-placeholder [id]="contentPlaceholder"></ec-template-placeholder>
    </div>
  </ng-template>
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

Notice that we now have the `#layoutTemplate` `<ng-template>`. Inside it is custom html markup that *includes placeholders* for the heading and content
templates. We have inserted a `<div>` between where the heading and the content would be placed to show the power of layout templates that could not be
accomplished with static markup mandated by the component.

Regarding the placeholders, we need to create a component for the purpose of marking where certain content would eventually be placed. We will address this
shortly.

## New Markup for Component with LayoutTemplate

We will now proceed to modify the html for our panel component:

```html
<ng-container *ngTemplateOutlet="layoutTemplate"> 
</ng-container>
```

Since we are now giving the implementor full control of the component's markup, we no longer need that top-level `<div>`.

But what happened to the `<ng-container>`s for the heading and content templates? To quickly address this concern, they will be placed via code instead of
markup, *including* any of the contexts that expose data to the front `<ng-template>`s.

## The Template Placeholder Component

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

It turns out that this template placeholder's other more important job, is to actually receive the appropriate template via the `TemplateRef<unknown>`
paramenter, along with the `context`, and `index` if applicable, instantiate template (which becomes an embedded view), and place the resulting DOM
"in its place" in the `#layoutTemplate`.

We have injected the `ViewContainerRef` service into this component's structure. `ViewContainerRef` makes the component to be able to be dynamic component
where we could load components and templates "into it" programmatically. The `createEmbeddedView` function of `ViewContainerRef` allows us to instantiate
a template, which may include the context and index (if its data were an array item). Note that we put double quotes around "into it", as in "to be able to be
a dynamic component where we could load components and templates 'into it'". `ViewContainerRef` doesn't really put components "in" existing components, but
rather as an immediate sibling of its resulting markup in html.

Now, we have exposed the `append()` and `clear()` functions as public because we will need to later dynamically insert template content from outside this
component later.

It is also very important that the template placeholder's markup is empty!

# Panel Component Modifications for #layoutTemplate

We will now need to make changes to our component code to recognize and work with the `#layoutTemplate`:

```javascript
export class PanelComponent implements OnInit {

  @Input() heading: string;

  @ContentChild('layoutTemplate', { read: TemplateRef }) layoutTemplate: TemplateRef<any>;
  @ContentChildren(TemplatePlaceholderComponent) layoutTemplateAsQueryList: QueryList<TemplatePlaceholderComponent>;

  @ContentChild('contentTemplate', { read: TemplateRef }) contentTemplate: TemplateRef<any>;
  @ContentChild('headingTemplate', { read: TemplateRef }) headingTemplate: TemplateRef<any>;

  headingPlaceholder: TemplatePlaceholderComponent;
  contentPlaceholder: TemplatePlaceholderComponent;

  isExpanded = true;

  constructor() { }
  
  ...
}
```

Let us start with the properties we will need to include.

We add a `@ContentChild` for the `#layoutTemplate` as we have with the content and heading templates.

We now have `@ContentChildren(TemplatePlaceholderComponent) layoutTemplateAsQueryList: QueryList<TemplatePlaceholderComponent>;` which "flags"
our panel component to be queryable for certain elements or components. We choose `@ContentChildren` to indicate that the component we would be looking for
is originally from the contents of an `<ng-template>`, that would finally become a part of this `PanelComponent`. Now that the content of `<ng-templates>`
would finally make it into our component, it is now safe to assume that we can query our component's own component tree for any descendant components, hence
`QueryList<ComponentType>`. Our component may have lots of elements and even components in it, and we can think of the generic parameter of `QueryList`
as a top-level filter that would automatically get us only the components of the specified `ComponentType`. In our case, we want to get a hold of the
two placeholders - the `headingPlaceholder` and `contentPlaceholder`, which are both of the type `TemplatePlaceholderComponent`. Once found, we would
then dynamically load the templates via their `append(..)` function.

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
      this.headingPlaceholder.append(this.headingTemplate, { heading: this.heading, toggleShowOrHide: this.toggleExpanded});
    }

    if (this.contentPlaceholder != null) {
      this.contentPlaceholder.clear();
      this.contentPlaceholder.append(this.contentTemplate);
    }
  }
```

We created a separate `loadTemplates()` function because its code block will be called from elsewhere besides in `ngDoCheck()`. Note that `ngDoCheck()` will
be called even when those placeholders haven't been found yet, since it occurs early in the component's lifecycle before our component receives the contents from
`<ng-template>`s.

For some unknown reason, there are times where `ngDoCheck()` will not automatically fire when our component is first created. This is why we explicitly call upon
it from inside `ngOnInit()`, but, for another unknown reason, we'll have to call it from `setTimeout()` with a 0-ms delay. Since `ngDoCheck()` will be called
multiple times, we will need to clear the placeholders first, then re-append the templates.

### The ngAfterContentInit() Lifecycle Event

The `ngAfterContentInit()` lifecycle event happens right after our component has received the contents of any `<ng-template>`, but before they're rendered
on the screen. For our purposes, our `layoutTemplateAsQueryList` `QueryList<ComponentType>` becomes available at this stage, *however*, we still will need
to wait for the contents of the `<ng-template #layoutTemplate>` to be accessible:

```javascript
  ngAfterContentInit() {
    this.layoutTemplateAsQueryList.changes.subscribe(c => {
      this.headingPlaceholder = this.layoutTemplateAsQueryList.find(i => i.id === 'headingPlaceholder');
      this.contentPlaceholder = this.layoutTemplateAsQueryList.find(i => i.id === 'contentPlaceholder');
      this.loadTemplates();
    });
  }
```

`QueryList<ComponentType>` has a `changes` property `Observable` that we can subscribe to once it finds all the components of type `ComponentType` in
our entire component, *including* the ones *outside* the `<ng-container *ngTemplateOutlet="layoutComponent">`. This is why we created an `@Input() id`
on the `TemplatePlaceholderComponent` so we can identify it in the `<ng-template>` upfront, and find it by id at this stage. This is where we load
the `headingPlaceholder` and `contentPlaceholder` references, and finally call `this.loadTemplates()` again.

If we run the application now, our panel will display as expected, according to the `#layoutTemplate`.

### Finishing Up

If we try to expand/contract the content, it doesn't seem to work. Recall that we have removed the `<ng-container *ngTemplateOutlet="contentTemplate">`
markup altogether since we are now instantiating and placing templates by code, and the markup had the `*ngIf` for when the `isExpanded` value changes. Since
all of that markup is gone, we will now have to simulate that in code. We modify the `loadTemplates()` function:

```javascript
 loadTemplates() {
    if (this.headingPlaceholder != null) {
      this.headingPlaceholder.clear();
      this.headingPlaceholder.append(this.headingTemplate, { heading: this.heading, toggleShowOrHide: this.toggleExpanded });
    }

    if (this.contentPlaceholder != null) {
      this.contentPlaceholder.clear();
      if (this.isExpanded) {
        this.contentPlaceholder.append(this.contentTemplate);
      }
    }
  }
```

Now, we can utilize the `this.isExpanded` value to decide whether to *really* "draw" the content template!

## Next Steps

It was quite a feat to use Angular's templating features and some of its lifecycle event hooks to be able to pull layout templates! It was all worth it because
we can now at will "draw" any panel component and use it to maintain expanded/contracted state per instance.

There is one thing to take care of. We didn't get a hold of the `QueryList<ComponentType>`'s `change` subscription so we can unsubscribe from it. We can
resolve that later as it is a topic aside from layout templating.

There is another topic about templating we need to address, and that is how to gracefully handle when the implementor doesn't specify an `<ng-template>` tag
that we might expect. The way we handle that depends on the templated component, but we must consider what or what not to show on the screen should they not
provide one.





