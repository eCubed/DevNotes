# Angular Directives - Starting Behavioral Directives

As we have seen, behavioral directives are practically attributes we put on an element's or component's tag, and they alter or add behavior to those
elements or components. We will first start writing a trivial directive. Our approach is to first create the directive and then learn how to use them
in our markup.

## Creating a Trivial Directive

We will start with creating a directive that has one altering behavior - put a simple border on an element.

We run the command `ng g d embox`, which creates our starting directive. We add a few things to it.

```javascript
import { Directive, ElementRef } from '@angular/core';

@Directive({
  selector: '[appEmbox]'
})
export class EmboxDirective {

  constructor(private elementRef: ElementRef) {
    this.elementRef.nativeElement.style.border = 'solid 1px black';
  }
}
```

We inject an `ElementRef` to the directive's constructor so that we have a hold on the top-level DOM element of the element or component that this
directive might be applied to.

We are directly manipulating DOM elements here in this example. More specifically, we are explicitly setting some CSS with

`this.elementRef.nativeElement.style.border = 'solid 1px black';`

Note that when we created this directive in the CLI, it got added to the `declarations` array in `app.module.ts`, just like components.

## Using the Directive

We can now use this directive anywhere in our app, using the selector `appEmbox` declared in our directive's `@Directive` decorator.

```html
<p appEmbox>
  This is a regular p tag that we applied a directive to
</p>
```

Note, though, that the effect of this directive looks equivalent to creating a CSS class with the rule `border: solid 1px black;`, and then
applying that class to the `p` tag. So, why bother with directives? Remember that this is just a trivial example. Remember that we can use this
even trivial directive everywhere in our app, so we can apply it to any element and we'd see the effect. So, what's the advantage? We did not
need to create many identical CSS classes in many components' .scss files.

## Passing in Values

So, we get a static black border, which isn't too flexible. We will need to make our directive take in a custom value. Let's allow users to
set the color. We modify our directive to take in an `@Input`:

```javascript
import { Directive, ElementRef, Input, OnInit } from '@angular/core';

@Directive({
  selector: '[appEmbox]'
})
export class EmboxDirective implements OnInit {

  @Input('border-color') borderColor: string;

  constructor(private elementRef: ElementRef) {
  }

  ngOnInit() {
    const borderColorToUse =  this.borderColor || 'black';

    this.elementRef.nativeElement.style.border = `solid 1px ${borderColorToUse}`;
  }
}
```

First, we create an `@Input` so that we can later specify an accompanying attribute in the markup we've applied this directive to. Notice that we
spelled out the exact attribute name `border-color` different from `borderColor`, the local variable name relative to this directive.

Also, notice that we have moved the CSS manipulation code to `ngOnInit`. Why? Because the constructor executes first before we have a change to
even read anything via `@Input`.

We also went ahead and handled the case had the markup not specified `border-color`. We'll take in whatever `border-color` value they pass in, if
they did. If not, we'll take `black` as the default value.

Now, we use this in the markup:

```html
<p appEmbox>
  directives-tests works!
</p>
<p appEmbox border-color="red">
  This is the directive with an accompanying property attribute
</p>
```

The first paragraph will have a black box around it, and the second paragraph will be surrounded by a red box.

Note that our attribute `@Input` property in the markup had no square brackets around it. Without square brackets around the attribute means that
we will take in the attribute value *as a string value*, or exactly the value as put in the markup.

With square brackets, the equivalent would be:

```html
<p appEmbox [border-color]="'red'">
  This is the directive with an accompanying property attribute
</p>
```

Say what? When we surround the attribute with square brackets, that makes whatever is between those double quotes *an expression* instead of a
literal string value. In our case, `'red'` with the single quotes *is* an expression that ultimately evaluates to a string whose value is `red`.
More often, the expression will be a reference to a property we set in the hosting component. For example:

```html
<p appEmbox border-color="borderColorPerHostComponent">
  This is the directive with an accompanying property attribute
</p>
```

```javascript
...
export class SomeHostComponent {
  borderColorPerHostComponent = 'red';

  constructor() { }
}
```

## When Attribute Name Equals DirectiveName

In order to apply a directive that takes in values, we both have had to specify the directive itself plus another attribute to take in a value.
This becomes cumbersome in the markup when the directive grows complex enough to take in more and more `@Input` values. It would be nice if we
could use only the directive's name in the markup and supply an `@Input` for it whose value is an actual object. So, for our example,
it would be nice to declare:

```html
<p [appEmbox]="borderColorPerHostComponent">
  This is the directive with an accompanying property attribute
</p>
```

The key to this is to create an `@Input` whose name is equivalent to the directive's selector name:

```javascript
export class EmboxDirective implements OnInit {

  @Input('appEmbox') borderColor: string;

  constructor(private elementRef: ElementRef) {
  }

  ngOnInit() {
    const borderColorToUse =  this.borderColor || 'black';

    this.elementRef.nativeElement.style.border = `solid 1px ${borderColorToUse}`;
  }
}
```

We merely changed `@Input('border-color')` to `@Input('appEmbox')`. This way, the directive itself can double up as a directive declaration
*and* an `@Input` attribute. This is a common approach.


