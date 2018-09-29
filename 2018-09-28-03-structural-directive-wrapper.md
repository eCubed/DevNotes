# Structural Directive as Wrapper to Template Content

Let's remember that structural directives are ultimately for DOM manipulation, and it's got to be more than just deciding whether or not to add the template's DOM
to the `ViewContainerRef`. With this in mind, we can now create a structural directive that, at the end, wraps some html around the directive's template. While we
found out that `TemplateRef` and `ViewContainerRef` by themselves cannot just directly manipulate the DOM, the only way to pull this is to first create a
component, yes, component that can take in a template and add it to its DOM.

## Reworking the Wrapper Component
In the previous article, we have the `EmpanelComponent`, which has the `<ng-content>` tag in it that actual content would eventually replace. We will need to
add some capabilities to this component so that we can programmatically set the template (in Typescript) as opposed to hard-coding it in markup.

For many of us, here is a new fact: `@Input()` can also be of type `TemplateRef<any>`, to which we can pass in an actual template reference to a component. We
previously thought that `@Input()` can only work with data objects.

Empanel Component Reworked:

```javascript
import { Component, OnInit, Input, TemplateRef } from '@angular/core';

@Component({
  selector: 'lib-empanel',
  templateUrl: './empanel.component.html',
  styleUrls: ['./empanel.component.scss']
})
export class EmpanelComponent implements OnInit {

  @Input() title: string;
  @Input() template: TemplateRef<any>;

  constructor() { }

  ngOnInit() {
    if (!this.title)
      this.title = "Default Heading";
  }
}
```

We now have an `@Input()` property called `template` of type `TemplateRef<any>`. In order to use that, we will need to modify the html for this component:

```html
<div class="panel">
  <h3>{{ title }}</h3>
  <div>
    <ng-container *ngTemplateOutlet="template"></ng-container>
    <ng-content></ng-content>
  </div>
</div>
```

Note that we keep the `<ng-content>` tag in case we still want to explicitly markup the component instead of using the structural directive.

We now have an `<ng-container>` tag, which, like `<ng-content>`, is an element-less placeholder for eventual content. It is different from `<ng-content>` 
by the capability of allowing us to specify a template right there in the markup whose content would eventually replace it. It applies the *ngTemplateOutlet*
directive, an Angular out-of-the-box directive, which points to a `TemplateRef<any>` whose content will eventually be put in place of the `<ng-container>`.

Before we proceed, we will need to add our component to a module's `entryComponents` array. If we're developing in a library, we would add this component to
that library's module declaration. Otherwise, we would add it to the module declaration of `app.module.ts`:

```javascript
@NgModule({
  imports: [
    CommonModule
  ],
  declarations: [
    ...,
    EmpanelComponent
  ],
  exports: [
    ...,    
    EmpanelComponent
  ],
  entryComponents: [
    EmpanelComponent
  ]
})
```

We have to import the `CommonModule` to recognize `*ngTemplateOutlet`. We need to put `EmpanelComponent` on the `entryComponents` array because we know that 
it will be dynamically instantiated in code (as opposed to us explicitly marking it up in a host component).

We are about 1/3 of the way done with the structural directive wrapper effort. The rest of the work is done in the wrapping structural directive.

## Creating a Wrapping Structural Directive

A wrapping structural directive's primary responsibility is to wrap its template with some predefined html. That predefined html *is* exactly a wrapper component.
The wrapping structural directive will actually instantiate a wrapper component, add it to the DOM, and then feed its template to the wrapper component.

We have seen that structural directives need the `TemplateRef<any>` and `ViewContainerRef` to "draw" the DOM. We now need a `ComponentFactoryResolver` whose
job is to create an instance of the component we need. We inject `ComponentFactoryResolver` to our structural directive's constructor.

We will first present the entire code and go through the most important parts.

```javascript
import {
  Directive,
  Input,
  ViewContainerRef,
  ComponentRef,
  OnInit,
  ComponentFactoryResolver,
  TemplateRef } from '@angular/core';
import { EmpanelComponent } from './empanel/empanel.component';

@Directive({
  selector: '[libEmpanel]'
})
export class EmpanelDirective implements OnInit {
  private title: string;

  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainerRef: ViewContainerRef,
    private componentFactoryResolver: ComponentFactoryResolver
  ) { }

  ngOnInit() {

    const factory = this.componentFactoryResolver.resolveComponentFactory(EmpanelComponent);
    let wrapperComponentRef = this.viewContainerRef.createComponent(factory);
    wrapperComponentRef.instance.title = this.title;
    wrapperComponentRef.instance.template = this.templateRef;
  }

  @Input() set libEmpanel(title: string){
    if (!title)
      this.title = 'title';
    else
      this.title = title;
  }
}
```

Let's start with something familiar - the `@Input() set` function. We'll want this because we still want to be able to specify a title to display on the heading
of the panel. Its logic will default to 'title' if the incoming value is null or undefined.

We also injected the `ComponentFactoryResolver` to the constructor. For now, the important thing we need to know about it is it creates a factory that knows how
to instantiate a component for us.

Since we are always going to want to draw the UI for this directive, we can put the building code in the `ngOnInit()` function, so the building happens automatically
when the structural directive is applied.

We first get a hold of the factory that will create an instance of the component for us:

```javascript
const factory = this.componentFactoryResolver.resolveComponentFactory(EmpanelComponent);
```

We pass to it exactly the class name of the wrapper component.

We actually do NOT explicitly create an instance by manipulating the factory ourselves. Instead, we pass it to the `ViewContainerRef`'s `createComponent()`
function. The `ViewContainerRef` itself will be the one to call on the factory to instantiate our component, and then add the component to the DOM. 
`createComponent()` also returns to us a reference to the instantiated component, which we can then manipulate:

```javascript
let wrapperComponentRef = this.viewContainerRef.createComponent(factory);
wrapperComponentRef.instance.title = this.title;
wrapperComponentRef.instance.template = this.templateRef;
```

Our component `EmpanelComponent` has two `@Input()` properties - title (string), and template (`TemplateRef<any>`). We programatically set those values
from the structural directive's code. It may feel strange because we are now doing in code what we used to do in markup. For example,
`wrapperComponentRef.instance.title = this.title` is the code equivalent of `<wrapper-component [title]="someTitle"></wrapper-component>`. Even more
strange is that we feed the template ref of our directive to the instantiated component in code with `wrapperComponentRef.instance.template = this.templateRef;`.
That is the code equivalent of `<wrapper-component ...><div>... arbitrary content ...</div></wrapper-component>` where templateRef refers to that div tag
placed inside the wrapper component tag.

## Next Steps

We have just achieved the basics of how to wrap html around a structural directive's template. This relieves the tedious work of manually creating the common html
markup (and CSS) on every component that we can think of that would want to be wrapped by that html. This approach is also flexible because we can apply the wrapping
to any element and component whenever we need to.

More complicated wrapper directive and component pairs would require some positioning programmatic positioning, such as popups and dropdowns. We will need to get
a hold of the directive's template's root element to obtain its dimensions and position so that we can calculate the position of what it is we want to pop out.
We'll explore some more of that type of nitty-gritty DOM and style manipulation in a later article.