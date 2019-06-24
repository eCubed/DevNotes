# Angular Templating - Custom Layout Components

In the previous article, 2019-06-09-06, we have updated our component to optionally be able to take in `TemplateRef`s via `@Input`s. Our component
so far is very robust and highly customizable. However, there is a small (but actually major) inconvenience. Though we have default templates to fall
back on when a template isn't specified (either via content children or `@Input`), we are *stuck* with the basic layout and look of our template - a
single div that wraps the heading and content templates, along with the CSS specified for our component. Yes, we get to define the look, but *only*
within each template. However, we are stuck with whatever html and CSS our component "put in stone". Suppose that we specified a padding and a general
background color for that wrapping div. Then, no matter how customized our heading and content templates are, we are stuck with our component's padding
and background color no matter what app we'd import our component into. That is a problem. That's like saying that if we imported a date picker library
into our projects, we're stuck with the look of the calendar decided by the author. This is what happens with most third party components - we're stuck
with the look.

In this article, we will explore how we can create components that will allow customizability of the top-level layout, not just per-template!

# Preparing our Component for Custom Layout Preparation

We know that we have two templates - heading and content, and we have also specified the `@Input` option and the default fallback template for each
of them. Our component also hard-codes the html and CSS that decides where to place our templates, which we're stuck with. We will keep the hard-coded
html and CSS, but now, we must regard that as a default layout template. So now, we modify our component's markup by surrounding our layout with
`<ng-template #defaultLayoutTemplate>`:

```html
<ng-template #defaultLayoutTemplate>
  <div class="panel">
    <div>--- Default Layout Template ---</div>
    <ng-container
      *ngTemplateOutlet="resolveHeadingTemplate(); context: createHeadingTemplateContext() ">
    </ng-container>
    <ng-container *ngIf="isExpanded">
      <ng-container *ngTemplateOutlet="resolveContentTemplate()">
      </ng-container>
    </ng-container>
  </div>
</ng-template>

<ng-template #defaultHeadingTemplate let-toggle="toggleShowOrHide">
  <h3 (click)="toggle()">{{ heading }}</h3>
</ng-template>
<ng-template #defaultContentTemplate>
  You have not supplied a #contentTemplate. Please supply one to replace this message.
</ng-template>
```

Because we surrounded our layout with an `<ng-template>` our entire component will not show. We know that it must show under a certain condition,
and we'll address that throughout this article. For now, we need to make a mental note that our template will somehow expect a custom layout. If it
exists, we'll apply it. If not, we show our default layout template. So, let's go ahead and add the markup that takes a layout into consideration:

```html
<ng-container *ngIf="panelLayout != null; else defaultLayoutTemplate">
  <ng-container *ngTemplateOutlet="panelLayout.rootTemplate"></ng-container>
</ng-container>

<ng-template #defaultLayoutTemplate>
  <div class="panel">
    <div>--- Default Layout Template ---</div>
    <ng-container
      *ngTemplateOutlet="resolveHeadingTemplate(); context: createHeadingTemplateContext() ">
    </ng-container>
    <ng-container *ngIf="isExpanded">
      <ng-container *ngTemplateOutlet="resolveContentTemplate()">
      </ng-container>
    </ng-container>
  </div>
</ng-template>

<ng-template #defaultHeadingTemplate let-toggle="toggleShowOrHide">
  <h3 (click)="toggle()">{{ heading }}</h3>
</ng-template>
<ng-template #defaultContentTemplate>
  You have not supplied a #contentTemplate. Please supply one to replace this message.
</ng-template>
```

For now, it looks like our component will have a `panelLayout` property, and if it doesn't exist, we show `#defaultLayoutTemplate`. If it did,
we go ahead and put the display the contents of `panelLayout` with `<ng-container *ngTemplateOutlet="panelLayout.rootTemplate"></ng-container>`.
What may not make sense now is `panelLayout.rootTemplate`. We will get to that later in this article. For now, we know that we *will need to create*
some class that manages our component's layout, and it has a `rootTemplate` property which *is actually* a `TemplateRef`. At this point, our component
is incomplete, since we will need to write some code on our component to recognize custom layouts.

## The Layout Base Class

It may not seem apparent now, but we will need to create a base (abstract) layout class that defines the required elements that *must be present* on a
custom layout. This base layout class will also expose functions that would allow the outside to add templates to it. Yes, our layout class will perform
dynamic template rendering, which would simulate `<ng-content *ngTemplateOutlet>` that we normally have in the default layout markup. Right now, we
present the entire code for `PanelLayoutBase`:

```javascript
export abstract class PanelLayoutBase {
  @ViewChild('rootTemplate', { read: TemplateRef }) rootTemplate: TemplateRef<any>;
  @ViewChild('headingInsertionPoint', { read: ViewContainerRef }) headingInsertionPoint: ViewContainerRef;
  @ViewChild('contentInsertionPoint', { read: ViewContainerRef }) contentInsertionPoint: ViewContainerRef;

  setHeading(headingTemplate: TemplateRef<any>, context: any) {
    this.headingInsertionPoint.createEmbeddedView(headingTemplate, context);
  }

  setContent(contentTemplate: TemplateRef<any>) {
    this.contentInsertionPoint.createEmbeddedView(contentTemplate);
  }

  clearContent() {
    this.contentInsertionPoint.clear();
  }
}
```

Let's focus on the declarations first. We see `@ViewChild`-decorated properties. This tells us automatically that later, we will create *actual components*,
specifically **layout components**, whose markups *must have* tags hash-identified with the identifiers stated in the `@ViewChild` decorator. For our case,
the actual layout component we'll need to write *will inherit* or in TypeScript, *extend* `PanelLayoutBase`, and its markup *must have* the three elements
tagged with `#rootTemplate`, `#headingInsertionPoint`, and `#contentInsertionPoint`.

Earlier, we stated that our layout component will need to supply in its markup tags labelled `#headingInsertionPoint` and `#contentInsertionPoint`.
Since those are actual placeholders we'll need to dynamically create template contents in their places, they will need to be typed `ViewContainerRef`,
and we will need to specify `{ read: ViewContainerRef }` since for some reason, Angular for now cannot infer the intent for a tag to be a `ViewContainerRef`.

We also expose functions that would allow a host to pass in templates and contexts to our layouts. Right now, we'll say that our Panel component will be the
entity that calls our layout's functions, and we'll address that later. These functions are wrappers for rendering templates via the insertion points'
`createEmbeddedView()` function. `ViewContainerRef`'s `createEmbeddedView()` takes in three arguments - first argument is the template, second argument
is the template context (optional), and the third is the index (also optional). Our wrapper functions should at least have the template parameter, and could
have the template context or index as necesary. In our case, the content template just displays content, so no context is required. The heading template
requires a context since we needed to expose the `isExpanded` state and the function to toggle that `isExpanded` state.

## Creating the Custom Layout Component

We now need to create a custom layout component, whose responsibility is to define the custom layout html and CSS. Let's start with the code:

```javascript
@Component({
  selector: 'app-custom-panel-layout',
  templateUrl: './custom-panel-layout.component.html',
  styleUrls: ['./custom-panel-layout.component.scss']
})
export class CustomPanelLayoutComponent extends PanelLayoutBase {

  constructor() {
    super();
  }
}
```

There is only one thing to do, and that is to extend the `PanelLayoutBase` class. This includes the requirement of calling `super()` from the constructor.
That is all we need to do. Now, let us write the markup:

```html
<ng-template #rootTemplate>
  <div class="custom-panel">
    <div>--- Custom Panel Layout ---</div>
    <ng-container #headingInsertionPoint></ng-container>
    <ng-container #contentInsertionPoint></ng-container>    
  </div>
</ng-template>
```

Earlier, we mentioned that all `@ViewChild`ren declared in `PanelLayoutBase` will need to have tags hash-identified accordingly, and here they are in the
concrete custom layout component's markup. The heading and content insertion points, which are `ViewContainerRefs` in code, are `<ng-container>`s in the
markup. This is so that there wouldn't be an extraneous element (most commonly a div) to encase them.

We will *always* need the top-level tag to be an `<ng-template #rootTemplate>`. This is *the only way* for our component to be able to display the custom
layout *entirely*. We will see how this is actually done shortly. For now, let us assume that our component will be able to access our layout's `rootTemplate`
and will be able to display it on the screen.

## Give the Component the Custom Layout

Let's jump to our component's usage momentarily:

```html
<ec-panel heading="My Custom Heading">
  <app-custom-panel-layout #layout></app-custom-panel-layout>
  <ng-template #headingTemplate let-heading="heading" let-showOrHide="toggleShowOrHide">
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

We see that we give our component a content child, which exactly *is* our custom component `<app-custom-panel-layout>`, tagged with the identifer `#layout`.
We'll use `#layout` for consistency and convention for depicting the custom layout for all templated components we'll create henceforth. Now, we have to 
create a `@ContentChild` declaration in our panel's markup so our component knows how to find it:

```javascript
export class PanelComponent implements AfterViewInit {

  @Input() heading: string;

  @ContentChild('headingTemplate', { read: TemplateRef }) headingTemplate: TemplateRef<any>;
  @Input('heading-template') headingTemplateFromInput: TemplateRef<any>;
  @ViewChild('defaultHeadingTemplate', { read: TemplateRef }) defaultHeadingTemplate: TemplateRef<any>;

  @ContentChild('contentTemplate', { read: TemplateRef }) contentTemplate: TemplateRef<any>;
  @Input('content-template') contentTemplateFromInput: TemplateRef<any>;
  @ViewChild('defaultContentTemplate', { read: TemplateRef }) defaultContentTemplate: TemplateRef<any>;

  @ContentChild('layout') panelLayout: PanelLayoutBase;
}
```

Note that the panel layout's class *is* the abstract layout base class, `PanelLayoutBase`, although we provided in our template's usage the concrete class.
From inside `PanelComponent`, we *don't care* what the actual concrete layout component is *as long as* it extends `PanelLayoutBase`. Thus, we will treat
the panelLayout *in an abstract manner*, meaning, we'll only call functions and properties that a `PanelLayoutBase` knows about.

## Managing the Panel Layout from Component

Since the panel layout handed to our component *has* `@ViewChild`ren, despite it being provided as a content child, we will only be able to manipulate the
panel layout at `ngAfterViewInit()`, not in `ngAfterContentInit()`:

```javascript
  ngAfterViewInit() {
    if (this.panelLayout != null) {
      this.panelLayout.setHeading(this.resolveHeadingTemplate(), this.createHeadingTemplateContext());
      this.panelLayout.setContent(this.resolveContentTemplate());
      this.changeDetectorRef.detectChanges();
    }
  }
```

We will need to check for existence of the `panelLayout` before proceeding to manipulate it. From this point, we call the layout's manipulation functions,
passing the appropriate templates and contexts. Most importantly, we *must* call `changeDetectorRef.detectChanges()`. We would need to inject the
`ChangeDetectorRef` service into our component's constructor for this to work.

## Afterthought

If we run our app now, we will see that our component *did* apply our custom component.

There's another niggling inconvenience. Though we can swap out layouts at design time (that is, provide explicitly the concrete layout component as a
content child), we still cannot specify an unknown layout type. We have worked on adding an option to be able to `@Input` templates. Can we `@Input` a
layout as well? It wouldn't be as easy, since we would need to pass in a type instead of a real object like `TemplateRef`. We'll explore this in the next
article.




