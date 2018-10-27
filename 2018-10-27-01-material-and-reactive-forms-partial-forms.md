# Angular Material and Reactive Forms - Partial Reactive Forms

We have previously discussed the situation when the model of our form has a property that is of an object type, and this property has many properties itself
that it probably calls for its own component to layout a *partial* form for it. Otherwise, our main form would get too crowded. We have created partial forms
as components before when working with template-driven forms. The partial form component would have an `@Input` where we would pass in a complex property of
an object, then that component would then be responsible for layout and binding.

We were previously concerned with how we're going to pull componentization when working with partial Reactive Forms because (1), the partial form cannot have
a form tag inside, and (2) we want to ensure that the partial form's local validations will be "heard" by its parents to determine the validity of the top-level
object. The cure to this problem is to have an `@Input()` property of type `FormGroup`. This may seem strange if we're used to the template-driven way of
passing models to components via `@Input`. Let us remind ourselves that when working with Reactive Forms, the descriptor of the form, or the `FormGroup` and
its hierarchy of reactive form objects is the major player here, not the model.

