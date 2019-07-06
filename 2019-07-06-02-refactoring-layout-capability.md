# Angular Templating - Refactoring Layout Capability

We have given our `Panel` component some custom layout capabilities. This included adding the `@Input`, `@ContentChild` and `@ViewChild`
`TemplateRef` properties as well as the `resolveLayoutTemplate()` function. It turns out that every component we'll write that we would want
the implementor to be able to totally physically cuspomize will have those properties and function. So, it is best to refactor a base class that
would take care of layout template functionality. We'll call it `TemplatedComponentBase`:

```javascript
export abstract class TemplatedComponentBase {

  @Input('layout-template') layoutTemplateInput: TemplateRef<any>;
  @ContentChild('layoutTemplate') layoutTemplate: TemplateRef<any>;
  @ViewChild('defaultLayoutTemplate') defaultLayoutTemplate: TemplateRef<any>;

  constructor() { }

  resolveLayoutTemplate() {
    return this.layoutTemplate || this.layoutTemplateInput || this.defaultLayoutTemplate;
  }

  abstract createLayoutTemplateContext(): any;
}
```

All of our templated components will extend this class and they will automatically have the layout properties and functions. Note, though, that
`createLayoutTemplateContext(): any` is abstract, since it will need to be upto our individual templated components to decide what to expose
(component values, functions, templates, and template contexts) to `<ng-template>`s.

We then modify our `Panel` component to inherit `TemplatedComponentBase`:

```javascript
export class PanelComponent extends TemplatedComponentBase implements OnInit {

  @Input() heading: string;
  @Input('is-expanded') isExpanded: boolean;

  @ContentChild('headingTemplate') headingTemplate: TemplateRef<any>;
  @Input('heading-template') headingTemplateFromInput: TemplateRef<any>;
  @ViewChild('defaultHeadingTemplate') defaultHeadingTemplate: TemplateRef<any>;

  @ContentChild('contentTemplate') contentTemplate: TemplateRef<any>;
  @Input('content-template') contentTemplateFromInput: TemplateRef<any>;
  @ViewChild('defaultContentTemplate') defaultContentTemplate: TemplateRef<any>;

  constructor() {
    super();
  }

  ngOnInit() {
    if (this.isExpanded == null) {
      this.isExpanded = true;
    }
  }

  toggleExpanded = () => {
    this.isExpanded = !this.isExpanded;
  }

  resolveHeadingTemplate() {
    return this.headingTemplate || this.headingTemplateFromInput || this.defaultHeadingTemplate;
  }

  resolveContentTemplate() {
    return this.contentTemplate || this.contentTemplateFromInput || this.defaultContentTemplate;
  }

  createHeadingTemplateContext = () => {
    return {
      heading: this.heading,
      toggleShowOrHide: this.toggleExpanded,
      isExpanded: this.isExpanded
    };
  }

  createLayoutTemplateContext() {
    return {
      isExpanded: this.isExpanded,
      createHeadingTemplateContext: this.createHeadingTemplateContext,
      headingTemplate: this.resolveHeadingTemplate(),
      contentTemplate: this.resolveContentTemplate()
    };
  }

  expand() {
    this.isExpanded = true;
  }

  contract() {
    this.isExpanded = false;
  }
}

```

No changes are necessary in our component's markup or wherever we plop an instance of our component on another [hosting] component.

So, creating a base class will result in less duplicate code among templated components, which is always good practice.