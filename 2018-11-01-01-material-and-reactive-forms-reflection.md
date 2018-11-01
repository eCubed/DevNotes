# Angular Material and Reactive Forms - Reflection

We have covered a lot of ground in the last week of exploring Angular Material and Reactive Forms. We nested reactive forms for a more complex object hierarchy. We also created
forms that reflect arrays of complex objects, including the ability, at runtime, to add, remove, and reorder items. We also created custom synchronous and asynchronous validators.
It does feel like we have covered enough to create just about any form. As always, there is always more to learn.

## Next Steps with Angular Material

We have covered a few material components, like `mat-form-field` and `mat-tool-bar` and their related components. Of course, there are more, and we haven't
covered them yet. We didn't cover drop-down lists, tables, and date-picker just to name a few. It is also possible to create our own Material widgets that we
can place inside `mat-form-field`, and we may need to do that later. There is also the dialog, or modal window, that isn't very obvious to use. We will explore
these as we explore and learn more things.

## Next Steps with Reactive Forms

We have covered Reactive Forms extensively in the past week, and fortunately, there is little left of something new to learn about it. Obviously, we did not explore
every single class, interface, or some built-in validators at all from `@angular/forms`. Whatever we didn't cover would just be a matter of looking them up in
documentations.

One particular interesting thing we didn't really explore with reactive forms is arrays of primitive data, most practically, an array of strings. Yes, we would
use `FormArray` to describe it in a `FormGroup`, but we won't make a partial reactive form component for it because it isn't a complex data type. How would
we validate each item individually? How would we validate the whole array by examining all children's values to determine if the array is legit?

Whenever we introduced a new form to explore reactive forms, all of them were ALWAYS in "create mode". That is, we always started with an empty form and accepted
the default values we specified in the top-level `FormGroup` instantiation. We have not yet explored how to programmatically fill the form with data that we
either hard-coded, or more importantly, one we obtained from a GET Api Call. We'll need to explore how to set an object to the top-level `FormGroup`.