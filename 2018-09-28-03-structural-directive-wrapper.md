# Structural Directive as Wrapper to Template Content

Let's remember that structural directives are ultimately for DOM manipulation, and it's got to be more than just deciding whether or not to add the template's DOM
to the `ViewContainerRef`. With this in mind, we can now create a structural directive that, at the end, wraps some html around the directive's template. While we
found out that `TemplateRef` and `ViewContainerRef` by themselves cannot just directly manipulate the DOM, the only way to pull this is to first create a
component, yes, component that can take in a template and add it to its DOM.

## Reworking the Wrapper Component
In the previous article, we have the `EmpanelComponent`, which has the `<ng-content>` tag in it that actual content would eventually replace. We will need to
add some capabilities to this component so that we can programmatically set the template (in Typescript) as opposed to hard-coding it in markup.

For many of us, here is a new fact: `@Input()` can also be of type `TemplateRef<any>`, to which we can pass in an actual template reference to a component. We
previously thought that `@Input()` can only work with object values.

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

We now have an `<ng-container>` tag, which is an element-less placeholder for eventual content. It applies the *ngTemplateOutlet* directive, an Angular 
out-of-the-box directive, which points to a `TemplateRef<any>` whose content will eventually be put in place of the `<ng-container>`.

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

We have to import the `CommonModule` to recognize `*ngTemplateOutlet`. We need to put `EmpanelComponent` on the `entryComponents` array just because we
know that it will be dynamically instantiated in code (as opposed to us explicitly marking it up in a host component).

We are about 1/3 of the way done with the structural directive wrapper effort. The rest of the work is done in the wrapping structural directive.

## Creating a Wrapping Structural Directive