# Angular Material and Reactive Forms - Reflection

We have covered a lot of ground with Reactive Forms. We nested reactive forms for a more complex object hierarchy. We also created forms that reflect arrays of complex 
objects, including the ability, at runtime, to add, remove, and reorder items. We also created custom synchronous and asynchronous validators. It does feel like we have 
covered enough to create just about any form. As always, there is always more to learn.

## Next Steps

We have covered Reactive Forms extensively and fortunately, there is little left of something new to learn about it. Obviously, we did not explore
every single class, interface, or some built-in validators at all from `@angular/forms`. Whatever we didn't cover would just be a matter of looking them up in
documentations.

One particular interesting thing we didn't really explore with reactive forms is arrays of primitive data, most practically, an array of strings. Yes, we would
use `FormArray` to describe it in a `FormGroup`, but we won't make a partial reactive form component for it because it isn't a complex data type. How would
we validate each item individually? How would we validate the whole array by examining all children's values to determine if the array is legit?

Whenever we introduced a new form to explore reactive forms, all of them were ALWAYS in "create mode". That is, we always started with an empty form and accepted
the default values we specified in the top-level `FormGroup` instantiation. We have not yet explored how to programmatically fill the form with data that we
either hard-coded, or more importantly, one we obtained from a GET Api Call. We'll need to explore how to set an object to the top-level `FormGroup`.

So far, we have always ultimately associated a `FormControl` with a primitive input element. Reactive Forms knows about these standard input elements, and so,
the framework knows what to do when it sees `formControlName` on a standard `<input>` (or `textarea`) element. However, what if we create a custom component
or widget, still associated with a primitive piece of data, but we want it to be able to participate in `Reactive Forms` because ultimately, we want it to
contribute to the entire form's validation? For example, let's say that we create a component that we input simple time by clicking on up/down arrows for each
digit. While we'll keep the logic from inside this component that would automatically translate back-and-forth between UI and a numeric value, what would we
need to do so that we can add the attribute `formControlName` to our component, say `<app-time-component formControlName="startTime></app-time-component>`,
where in code, `startTime` is a `FormControl` initialized with the value 0 (to denote 12 AM). Reactive Forms doesn't know about our custom component, so
putting `formControlName` will cause an error.