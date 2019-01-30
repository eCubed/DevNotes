# Angular Directives - Primer

One of the more advanced topic in Angular is directives. Directives are a feature that allows us to affect an existing element or even a component we 
make, in some specific way. This would then free us from having to create properties and methods in our component that have anything to do with a
specific functionality other than manipulating the very data our components were originally and primarily created for.

There are at least two kinds of directives - structural and behavioral.

A *structural directive* is a DOM-building and/or a DOM-destructing directive. We have structural directives before - *ngIf* and *ngFor*, which are
both preceded by a * character. The leading * character denotes the fact that stuff will be added/removed from the DOM based on some logic. With *ngIf*,
we know that the component or element we apply it to will actually not exist in the DOM if the condition is false, not just making it merely invisible
by applying the CSS *display: none* on the element. The *ngFor* structural directive is designed to actually create a DOM tree for each item in an
array.

A *behavioral directive* does not actually add or remove the DOM of the component or element it applies to. Instead, it's designed to alter the behavior,
hence, its name. Most likely, though, especially when we start learning how to make our own directives, the *behavior* is to alter the look of the
element or component. A behavioral directive can also subscribe to services to receive continuously-changing data, and the ever-changing values
returned from those services can be used to alter the appearance of whatever component or element it was applied.

We will start with creating trivial behavioral directives, starting with those that don't take in values. We will later see how to properly create
directives that take in attribute values.