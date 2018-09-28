# Structural Directives - Diving In

I know that there are attribute directives that are typically used to alter appearance and behavior of elements and components that apply them. I am focusing on
structural directives - those that will actually construct or destruct html (ultimately) based on input values and other specific logic. So, I start with a trivial
structural directive:

```javascript
import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';

@Directive({
  selector: '[libTrivial]'
})
export class TrivialDirective {

  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainerRef: ViewContainerRef
  ) { }

  @Input() set libTrivial(someValue: string) {
    /* We are not even using someValue at this point, but this is a start.
    */
    this.viewContainerRef.createEmbeddedView(this.templateRef);
  }
}
```

And its usage:

```html
<p *libTrivial="'some value I want'">
  This is the content of my p-tag.
</p>
```

Like `*ngIf` and `*ngFor` that we've used, since they're structural directives, we also need to specify the asterisk before the directive selector name. This will
tell Angular that the element or component that applies the directive *becomes a template* of our directive. This may not make sense now, but when we code from
inside a structural directive, any element or component that applies our structural directive is referred to a template that we can later access and manipulate.

There are a few required pieces to constructing a structural directive:

`selector: '[libTrivial]'`. We need to name our directive with an identifier that we'll end up applying on elements and components.

`TemplateRef<any>`. We need to inject `TemplateRef<any>` - the handle to the very component or element that applies our structural directive.

`ViewContainerRef`. We need to inject `ViewContainerRef` - the governing entity to which we, at will, explicitly instruct to show (embed) or not show (clear)
the `templateRef`

`@Input() set libTrivial(someValue: string)`. We will need to include an `Input() set` function to our structural directive if we intend to take in some
value. Notice that the function's name (in our case, libTrivial) has to match the name of the directive as specified in the `selector` in the `@Directive` 
declaration. This is the very mechanism that allows us to to take in some value that would help our directive decide whether to show the template or not. 
Another important capability of the `@Input() set` is that our function will be run *every time* the incoming value changes value. No, we do not explicitly call this function in any piece of code.
The framework calls it for us automatically. Note, though, that in our application of this directive on a component or element, we can actually pass in functions and
hosting component variables (properties) as well, aside from a hard-coded value.

The example above doesn't do anything but unconditionally (just) display the element or component that applies our structural directive.

## Adding some Show or Hide Logic

Let's say that for our trivial structural directive to show the template, the string passed in must be exactly 'show me'. We'll need to code the following logic
to the `@Input() set` function:

```javascript
  @Input() set libTrivial(someValue: string) {
    
    if (someValue == 'show me')
      this.viewContainerRef.createEmbeddedView(this.templateRef);
    else
      this.viewContainerRef.clear();
  }
```

The `viewContainerRef.createEmbeddedView(templateRef)` function is what ultimately puts the template contents on the screen. If the condition (incoming value
must be equal to `show me`) is true, then we "draw" the template. If the input value is any other string, we clear the view container, in effect, actually physically
removes the DOM of the template, if it has any contents previously loaded by `createEmbeddedView`.

We may think "why bother putting the else statement and clearing it the view container?". We need to remember that the `@Input() set` function gets called 
automatically when the input value changes. If we don't ever clear the view container, every time the incoming value is 'show me', it will tack on another DOM of
our template, and doing nothing if the value is something else. We'd accumulate multiple instances of the template on the screen.

This is very similar to how the `*ngIf` directive works.

## An Even More Trivial Directive

I just realized that there can be a structural directive that doesn't have to have an `@Input() set` function. There will be no value passed in that would help
determine whether the template will show or not even be in the DOM (absolutely gone, not merely hidden). This is the case where our directive will ALWAYS
display the template no matter what.

```javascript
import { Directive, Input, TemplateRef, ViewContainerRef, OnInit } from '@angular/core';

@Directive({
  selector: '[libMoreTrivial]'
})
export class MoreTrivialDirective implements OnInit{

  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainerRef: ViewContainerRef
  ) { }

  ngOnInit() {
    this.viewContainerRef.createEmbeddedView(this.templateRef);
  }
}
```

We no longer have an `@Input() set` function. We now implement `OnInit`, and embed the template into the view container so that the template will show. So now,
we apply the directive to an element or component:

```html
<p *libMoreTrivial>
  This is the content of my p-tag.
</p
```

The application of the directive would then not need a value. This seems strange because all of the structural directives we've used (out-of-the-Angular-box) always
followed after them `="someValue"`.


## Next Steps

It seems like the only thing we'll be able to do in a structural component is practically to show or hide the template based on the incoming value. The official
docs say that structural directives "change the DOM layout by adding and removing DOM elements", which sounds a lot more than just showing and hiding the element
or component that applies our structural directive. We want to be able to sometimes wrap our template with other HTML, or maybe even supplant it to another parent
so we can achieve things like popups. Alas, the `TemplateRef<any>` and `ViewContainerRef` alone cannot perform those kinds of tasks. **Warning**: Both of them 
have functions that look like we can insert HTML elements. If we try them, we will NOT get compilation errors, but we will get runtime errors! 

