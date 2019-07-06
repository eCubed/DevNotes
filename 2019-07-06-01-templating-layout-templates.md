# Angular Templating - Custom Layout Components

In the previous article, we have updated our component to optionally be able to take in `TemplateRef`s via `@Input`s. Our component so far is very robust and highly 
customizable. However, there is a small (but actually major) inconvenience. Though we have default templates to fall back on when a template isn't specified (either via 
content children or `@Input`), we are *stuck* with the basic layout and look of our template - a single div that wraps the heading and content templates, along with the 
CSS specified for our component. Yes, we get to define the look, but *only* within each custom template. However, we are stuck with whatever html and CSS our component "set
in stone". Suppose that we specified a padding and a general background color for that wrapping div. Then, no matter how customized our heading and content templates are, 
we are stuck with our component's padding and background color no matter what app we'd import our component into. That is a problem. That's like saying that if we imported a
date picker library into our projects, we're stuck with the look of the calendar decided by the author. This is what happens with most third party components - we're stuck
with the look.

In this article, we will explore how we can create components that will allow customizability of the top-level layout, not just per-template!

# Preparing our Component for Custom Layout Preparation

We know that we have two templates - `#headingTemplate` and `#contentTemplate`, and we have also specified the `@Input` option and the default fallback template for each
of them. Our component also hard-codes the html and CSS that decides where to place our templates, which we're stuck with. We will keep the hard-coded html and CSS, but now,
we must regard that as a default layout template. So now, we modify our component's markup by surrounding our layout with
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

Because we surrounded our layout with an `<ng-template>` our entire component will not show if we attempt to run the app now. We know that it must show under a certain
condition, and we'll address that throughout this article. For now, we need to make a mental note that our template will somehow expect a custom layout. If it exists, we'll
apply it. If not, we show our default layout template. So, let's go ahead and add the markup that takes a layout into consideration:

```html
<ng-container *ngTemplateOutlet="resolveLayoutTemplate()"></ng-container>

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

Our component now has an `<ng-container>` where a layout (practically our entire component) will be displayed. We will let our component's code decide which layout to
feed to this `<ng-container>` the same exact way that we decide which (between content-child, input, or default) template to ultimately render. So, this means, we must
add layout `TemplateRef`s in our code as well as the function `resolveLayoutTemplate`:

```javascript
  @Input('layout-template') layoutTemplateInput: TemplateRef<any>;
  @ContentChild('layoutTemplate') layoutTemplate: TemplateRef<any>;
  @ViewChild('defaultLayoutTemplate') defaultLayoutTemplate: TemplateRef<any>;

  ...

  resolveLayoutTemplate() {
    return this.layoutTemplate || this.layoutTemplateInput || this.defaultLayoutTemplate;
  }
```

Just like the content and heading templates, we will let the implementor of our component provide a layout template via content child and input, as well as provide a default
layout in case they don't specify a layout.

## Layout Template Context

When we are giving the implementor full control of our component's layout, it means we will literally let them construct the markup and provide custom CSS for our entire
component, *from outside* our component. For our panel component, an external `<ng-template #layoutTemplate>` will need to know the `isExpanded` states, which "version"
of each of the other templates to render (content child provided, via input, or default), and a reference to the `createHeadingTemplateContext` function. So, just like
we wrote create template context functions for `#headingTemplate`, we'll also create one for our `#layoutTemplate`:

```
  createLayoutTemplateContext() {
    return {
      isExpanded: this.isExpanded,
      createHeadingTemplateContext: this.createHeadingTemplateContext,
      headingTemplate: this.resolveHeadingTemplate(),
      contentTemplate: this.resolveContentTemplate()
    };
  }
```

Now, back at our markup, we'll need to provide the context to `<ng-container *ngTemplateOutlet="resolveLayoutTemplate()">`:

```html
<ng-container *ngTemplateOutlet="resolveLayoutTemplate(); context: createLayoutTemplateContext()"></ng-container>
```

Now, any `<ng-template>` we'll regard as a layout template, whether from outside or component, *or even* `#defaultLayoutTemplate`, will now have access to the values
and functions from inside our component, so the layout template in question can utilize them for more enhanced customization!

But before we move on, note that we're also giving access to `createHeadingTemplateContext`, which *is* a function. Recall that exposing one of our component's internal
functions via template context (or via another object at all), would change the context of `this` wherever that function is called. Since `createHeadingTemplateContext`
could be called from outside our component, TypeScript will think that the `this` *is* the template context, not our component. So, to solve that, we will need to rewrite
`createHeadingTemplateContext` as an *arrow function*:

```javascript
  createHeadingTemplateContext = () => {
    return {
      heading: this.heading,
      toggleShowOrHide: this.toggleExpanded,
      isExpanded: this.isExpanded
    };
  }
```

Right now, our `#defaultLayoutTemplate` is fine, meaning, if no layout was specified from outside, it would display on our app with the expected values and results.
Notice that from inside `#defaultLayoutTemplate`, we are directly accessing our component's functions and properties, because we can. However, it is highly advisable,
though optional, that we should write `#defaultLayoutTemplate` *as if* it were an `<ng-template>` written *outside* our component. This means that 
`#defaultLayoutTemplate` *should access* the template context via `let-`, and use values and functions exposed via the template context instead of directly accessing
functions and values of our component:

```html
<ng-template #defaultLayoutTemplate let-headingTemplate="headingTemplate" let-createHeadingTemplateContext="createHeadingTemplateContext"
  let-isExpanded="isExpanded" let-contentTemplate="contentTemplate">
  <div class="panel">
    <div>--- Default Layout Template ---</div>
    <ng-container *ngTemplateOutlet="headingTemplate; context: createHeadingTemplateContext() ">
    </ng-container>
    <ng-container *ngIf="isExpanded">
      <ng-container *ngTemplateOutlet="contentTemplate">
      </ng-container>
    </ng-container>
  </div>
</ng-template>
```

This way, when it's time to use our component elsewhere, we can merely copy-paste his markup snippet onto our apps, and make a few modifications. This saves us a lot of
thinking and typing as utilizing template context values and functions is tedious and error-prone. The only things that we would need to modify is the template identifier,
the markup, and where we'll want to place those `<ng-container>` placeholders.

## Providing Layout Templates

We can provide layout templates in our component's markup on a host component via content child or via input. First, via content child:

```html
<lib-panel heading="My Custom Heading">
  <ng-template #layoutTemplate let-headingTemplate="headingTemplate"
    let-createHeadingTemplateContext="createHeadingTemplateContext" let-isExpanded="isExpanded"
    let-contentTemplate="contentTemplate">
    <div class="panel">
      <div>--- Custom Layout Template ---</div>
      <ng-container *ngTemplateOutlet="headingTemplate; context: createHeadingTemplateContext()"></ng-container>
      <ng-container *ngIf="isExpanded">
        <ng-container *ngTemplateOutlet="contentTemplate">
      </ng-container>
    </div>
  </ng-template>
  <ng-template #headingTemplate let-heading="heading" let-showOrHide="toggleShowOrHide">
    <div class="heading">
      <span class="heading-text">{{ heading }}</span>
      <span class="spacer"></span>
      <span class="close" (click)="showOrHide()">YY</span>
    </div>
  </ng-template>
  <ng-template #contentTemplate>
    Custom content
  </ng-template>
</lib-panel>
```

Now, via input:

```html
<lib-panel heading="My Custom Heading" [layout-template]="layoutTemplate">  
  <ng-template #headingTemplate let-heading="heading" let-showOrHide="toggleShowOrHide">
    <div class="heading">
      <span class="heading-text">{{ heading }}</span>
      <span class="spacer"></span>
      <span class="close" (click)="showOrHide()">YY</span>
    </div>
  </ng-template>
  <ng-template #contentTemplate>
    Custom content
  </ng-template>
</lib-panel>
<ng-template #layoutTemplate let-headingTemplate="headingTemplate"
  let-createHeadingTemplateContext="createHeadingTemplateContext" let-isExpanded="isExpanded"
  let-contentTemplate="contentTemplate">
  <div class="panel">
    <div>--- Custom Layout Template ---</div>
    <ng-container *ngTemplateOutlet="headingTemplate; context: createHeadingTemplateContext()"></ng-container>
    <ng-container *ngIf="isExpanded">
      <ng-container *ngTemplateOutlet="contentTemplate">
    </ng-container>
  </div>
</ng-template>
```

## Afterthought


