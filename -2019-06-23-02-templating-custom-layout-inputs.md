# Angular Templating - Custom Layout Inputs

We have discussed the need to let the implementor of our templated components specify a layout via `@Input` like we have with our component templates.
While the templates are actual `TemplateRef`s, we can easily pass them around via `@Input`.  We could plop down in our component's markup tags the 
content child the tag selector for our concrete layout. For the `@Input` equivalent of the content child way of giving a custom layout component to our
templated component, we would *not* be passing in instances of a fully-instantiated and available instance of a custom layout component. So, no, the
type of the input is NOT going to be that of a concrete layout component, but of a Type instead:

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

  @Input('layout-class') panelLayoutClassFromInput: Type<PanelLayoutBase>;
  @ViewChild('panelLayoutFromInputPlaceholder', { read: ViewContainerRef }) panelLayoutFromInputPlaceholder: ViewContainerRef;
}
```

We create an `@Input` for our layout of type `Type<PanelLayoutBase>`. This ensures that the class passed in *extends* `PanelLayoutBase`. Note that
we could have put `Type<any>` and our entire setup would work, but specifying the abstract type `PanelLayoutBase` looks more semantically settling.

Now, we will need to also put a `ViewContainerRef` placeholder for the instance of our custom panel layout component. We would be given a the class of
the component with `@Input('layout-class')`, and yes, we will need to dynamically create an instance of the panel layout component and put it in that
placeholder. Remember that our custom panel layout components should have a top-level `<ng-template #rootTemplate>`, so adding an instance of a
custom panel layout component won't render any markup on the screen, which is what we need. So, we also add an accompanying tag for this `ViewContainerRef`
on our component's markup:

```html
<ng-container *ngIf="panelLayout != null; else defaultLayoutTemplate">
  <ng-container *ngTemplateOutlet="panelLayout.rootTemplate"></ng-container>
</ng-container>

<ng-container #panelLayoutFromInputPlaceholder></ng-container>

<ng-template #defaultLayoutTemplate>
  <div class="panel">
    <div>--- Default Layout Template ---</div>
    <ng-container *ngTemplateOutlet="resolveHeadingTemplate(); context: createHeadingTemplateContext() ">
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

Now, we need to instantiate the panel layout component:

```javascript
 ngAfterViewInit() {

    if (this.panelLayoutClassFromInput != null) {
      const factory = this.componentFactoryResolver.resolveComponentFactory(this.panelLayoutClassFromInput);
      const componentRef = this.panelLayoutFromInputPlaceholder.createComponent(factory);
      this.panelLayout = componentRef.instance;
      this.changeDetectorRef.detectChanges();
    }

    if (this.panelLayout != null) {
      this.panelLayout.setHeading(this.resolveHeadingTemplate(), this.createHeadingTemplateContext());
      this.panelLayout.setContent(this.resolveContentTemplate());
      this.changeDetectorRef.detectChanges();
    }
  }
```

First, we detect to see if our component got a `panelLayoutClassFromInput`. If it did, then we go ahead and instantiate one. The first two lines of
code are about setting up and instantiating a component, which we won't discuss here since that is another topic. Note that we set `componentRef.instance`,
which is the instantiated custom layout component itself, to `this.panelLayout`. Now, at this point, `this.panelLayout` exist like it does had we
given our component a view child. We then call `this.changeDetectorRef.detectChanges()` for our component to recognize what we've done.

Now, how would we use this in markup?

```html
<ec-panel heading="My Custom Heading" [heading-template]="headingTemplateToInput"
  [layout-class]="getPanelLayoutClass()">
  <ng-template #contentTemplate>
    Custom content
  </ng-template>
</ec-panel>
```

It seems like we could have just written `[layout-class]="CustomPanelLayoutComponent"`, but we cannot do that since Angular would look for a property
in our class named exactly `CustomPanelLayoutComponent`. Instead, we write the function that returns to us the class:

```javascript
  getPanelLayoutClass(): Type<PanelLayoutBase> {
    return CustomPanelLayoutComponent;
  }
```

When we run the app, our component will recognize the layout class passed to its `@Input('layout-class')` property, and we will see that our component
instantiated the right class and finally rendered our custom template!