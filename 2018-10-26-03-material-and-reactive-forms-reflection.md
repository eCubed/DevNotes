# Angular Material and Reactive Forms - Reflection

So far, we have laid out a form nicely using Angular Material and Flex Layout. We have also worked with Reactive Forms alongside Angular Material. We were able to
explore form validation with Reactive Forms, and applied them using framework-built-in validators. We have also briefly discussed more advanced validation such as
custom validators and validators that may call upon an external API to determine whether the input valud is valid. At this point, we will place validation on hold
until a new one is necessary.

There are quite a bit more to explore in Reactive Forms especially pertaining to data. Since we can extract an object from a `FormGroup`, we may suspect that it
has to be possible to extract an object with complex properties. In real life, it is often a requirement that a model off a form have properties that are objects
themselves. How would we set up the top-level `FormGroup` to reflect a complex property? It turns out that it is possible to compose a hierarchy of `FormControls`
and `FormGroups`. The only somewhat undesirable consequence of this, is it would crowd our markup especially if it's more than 2 object hierarchy levels deep.
We'll explore models with complex objects.

Another special case with objects is that some of them have arrays - both of primitive values and object types. We know that we can easily reflect object hierarchies
with building `FormGroups` and `FormControls`, but how would be build a form descriptor for a model that requires an array of an object? Would it be possible to
validate against the array, like, it cannot exceed a certain amount? Will we need to write our own validators for arrays?

What if the object has a complex-enough property that it warrants for it to have its own component, and we still want that component to act like a Reactive Form?
I don't think this component can have a form tag inside itself because this would mean that we would be nesting form tags, which is not allowed. Wouldn't we need
to make the component know how to first detect its own validity and then let some parent know somehow when it's in an invalid state? How would we pull this!!!?
I'm currently checking out [this article](https://itnext.io/partial-reactive-form-with-angular-components-443ca06d8419) to see if it will answer my questions.
The key is to pass a `FormGroup` to the child component, and then have the child component manipulate it? And supposedly, the parent component's own `FormGroup`
would automatically hear anything about validation from the child components.
