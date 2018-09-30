# Structural Directive as Wrapper to Template Content

Let's remember that structural directives are ultimately for DOM manipulation, and it's got to be more than just deciding whether or not to add the template's DOM
to the `ViewContainerRef`. With this in mind, we can now create a structural directive that, at the end, wraps some html around the directive's template. While we
found out that `TemplateRef` and `ViewContainerRef` by themselves cannot just directly manipulate the DOM, the only way to pull this is to first create a
component, yes, component that can take in a template and add it to *its* DOM.

## Registering a Component to be Dynamically Instantiable

We currently have the `EmpanelComponent` from the previous article. It is ready to be used as it is. There are no modifications needed for both code and markup.
However, the component is not dynamically instantiable until we register it to be. This is done by including it in an `@NgModule` declaration's `entryComponents`
array. If we're developing in a library, we would add this component to that library's module declaration. Otherwise, we would add it to the module declaration of
`app.module.ts`:

```javascript
@NgModule({
  imports: [
    ...
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
Now, we can dynamically instantiate a component. We will perform that in the structural directive.

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
    const wrapperComponentRef = this.viewContainerRef.createComponent(factory,
      0, undefined, [ this.templateRef.createEmbeddedView({}).rootNodes ]);
    wrapperComponentRef.instance.title = this.title;
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

We also injected the `ComponentFactoryResolver` to the constructor. For now, the important thing we need to know about it is it creates a factory, and that factory
knows how to instantiate a component for us.

Since we are always going to want to draw the UI for this directive, we can put the building code in the `ngOnInit()` function, so the building happens automatically
when the structural directive is applied.

We first get a hold of the factory that will create an instance of the component for us:

```javascript
const factory = this.componentFactoryResolver.resolveComponentFactory(EmpanelComponent);
```

We pass to it exactly the class name of the wrapper component.

We actually do NOT explicitly create an instance by manipulating the factory ourselves. Instead, we pass it to the `ViewContainerRef`'s `createComponent()`
function. The `ViewContainerRef` itself will be the one to call on the factory to instantiate our component, and then add the component to the DOM. 

```javascript
const wrapperComponentRef = this.viewContainerRef.createComponent(factory,
  0, undefined, [ this.templateRef.createEmbeddedView({}).rootNodes ]);
```
* `createComponent()`'s first parameter is the factory so that `viewContainerRef` will be able to obtain an instance of our component for us.
*  The second parameter is the index where we'll want to place the the newly-instantiated component in the DOM. In our case, the component is the only one to be
dynamically created.
*  The third parameter is an Injector object. We are not using one for wrapping a panel around content. We will later see that we will want to specify an object
in this third parameter that Angular would inject to its constructor when the component is instantiated.
*  The fourth parameter is a double array that make up exactly the DOM elements we would want to transclude to the dynamically-instantiated component, the exact
elements that would put in place of the component's `<ng-content>` tag. In our case, we want to pass the directive's template contents to our component so that
it can substitute them for its `<ng-content>` tag. In order to get the elements of the templateRef, we call its `.createEmbeddedView()` function, pass to it an
empty object (really, a context, but we're not using one right now), and then access the `rootNodes` property. `rootNodes` is a single array, so we wrap it inside
square brackets so that it's a double array of one item, and that first and only elements have all those DOM elements.

`createComponent()` also returns to us a reference to the instantiated component, which we can then manipulate:

```javascript
wrapperComponentRef.instance.title = this.title;
```

Our component `EmpanelComponent` has the `@Input() title: string` property. We programatically set it from the tructural directive's code. It may feel strange
because we are now doing in code what we used to do in markup. For example, `wrapperComponentRef.instance.title = this.title` is the code equivalent of 
`<wrapper-component [title]="someTitle"></wrapper-component>`.

## Next Steps

We have just achieved the basics of how to wrap html around a structural directive's template. This relieves the tedious work of manually creating the common html
markup (and CSS) on every component that we can think of that would want to be wrapped by that html. This approach is also flexible because we can apply the wrapping
to any element and component whenever we need to.

More complicated wrapper directive and component pairs would require some positioning programmatic positioning, such as popups and dropdowns. We will need to get
a hold of the directive's template's root element to obtain its dimensions and position so that we can calculate the position of what it is we want to pop out.
We'll explore some more of that type of nitty-gritty DOM and style manipulation in a later article.