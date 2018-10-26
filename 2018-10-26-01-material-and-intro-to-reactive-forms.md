# Angular Material and Introduction to Reactive Forms

In this series, we will adapt Angular Reactive Forms to use in conjunction with Angular Material. This article will also serve as an introduction to Reactive Forms,
which is a more intricate and powerful way to manage form data than template-driven forms (where we used `[(ngModel)]`). With Reactive Forms, we are given tools to
make form and per-input validation easier to develop, and other featuers.

## Breaking Away from Template-Driven Forms

If we are coming from developing with template-driven forms, we are used to storing a model in a component, and then binding its properties to various elements 
declaring `[(ngModel)]="model.property"`. This is NOT the game plan in Reactive Forms. For one, we will NOT be storing an instance of a model in our component.
We will also NOT be using `[(ngModel)]` to bind model property values to a specific element. Instead, in Typescript code, we will declare a special object that
will define *the form* itself. To define the form, we will need to declare what form elements it will have. And with each form element declared, we also can declare
validations, or rules that define accepted values for that form control. On the markup, we associate our form with that special object, and then associate each
element with the corresponding declared form element.

For now, we just want to condition ourselves to break away from the `ngModel` way of working with forms. For now, it is important to know and embrace the fact that
we will be describing the very form itself in Typescript code, and we will match declared form controls with the actual form elements in the markup. We will see this
in action later.

## Beginning Form

We're working on a very simple registration form. It's submission is not functional because we want to focus on Reactive Forms concepts. The form will ask for 3
things - first name, last name, and email address. Naturally, these three fields call for a simple text box to collect data. We start with the following html:

```html
<form>
  <div fxLayout="row" fxLayout.lt-md="column" fxLayoutFill="space-between">
    <div fxFlex="0 1 calc(30% -10px)" fxLayout="column" >
      <mat-form-field>
        <input matInput type="text" placeholder="First Name" />
      </mat-form-field>
      <mat-form-field>
        <input matInput type="text" placeholder="Last Name" />
      </mat-form-field>
      <mat-form-field>
        <input matInput type="text" placeholder="Email" />
      </mat-form-field>
      <button mat-raised-button color="primary" type="submit">Submit</button>
    </div>
    <div fxFlex>
    </div>
    <div fxFlex="0 1 calc(30% - 10px)">
      The JSON of the registration info will be put here later.
    </div>
  </div>
</form>
```

We went ahead and put Material components and Flex Layout directives. We did not yet put anything specific to Reactive Forms.

## Declaring the Form in Typescript
Right now, we know that we want a form that takes in 3 string values. We also know in our heads, and ultimately, we are working on a model with 3 properties, each
with a string type. We are ready to work with `FormGroup` and `FormControl`.

```javascript
...
import { FormGroup, FormControl } from '@angular/forms';

@Component({ ... })
export class RegistrationComponent implements OnInit {

  registrationForm: FormGroup;

  constructor() { }

  ngOnInit() {
    this.registrationForm = new FormGroup({
      firstName: new FormControl(''),
      lastName: new FormControl(''),
      email: new FormControl('')
    });
  }
}
```

On `ngOnInit()` we declare what our form looks like. The form is represented by a special object called a `FormGroup`. The constructor of `FormGroup` takes in
an object whose properties are explicitly declared by us, and each property *is* an instance of a `FormControl`. We can name the properties of the object anything
we want, but by convention, the name would be *exactly* the same as the name of the model's property. We will want to stick to this naming scheme because these
property names declared here will be the exact property names of the model we'll later extract from the top-level `FormGroup`.

The `FormControl` object's constructor first takes in a parameter that is exactly the initial value we want. It has additional parameters (like validation) that
we'll later consider. Typically, we'll initialize a form to have empty strings.

## Associating `FormGroup` with the Form in the Markup
Now we are ready to associate the `FormGroup` with our form, along with matching up `FormControls` with the corresponding input elements in the markup:

```html
<form [formGroup]="registrationForm">
  <div fxLayout="row" fxLayout.lt-md="column" fxLayoutFill="space-between">
    <div fxFlex="0 1 calc(30% -10px)" fxLayout="column" >
        <mat-form-field>
          <input matInput type="text" formControlName="firstName" placeholder="First Name" />
        </mat-form-field>
        <mat-form-field>
          <input matInput type="text" formControlName="lastName" placeholder="Last Name" />
        </mat-form-field>
        <mat-form-field>
          <input matInput type="text" formControlName="email" placeholder="Email" />
        </mat-form-field>
        <button mat-raised-button color="primary" type="submit">Submit</button>
    </div>
    <div fxFlex>
    </div>
    <div fxFlex="0 1 calc(30% - 10px)">
      <pre>{{ registrationForm.value | json }}</pre>
    </div>
  </div>
</form>
```

We associate the form in the markup with the `FormGroup` object declared in Typescript with the `[formGroup]` directive in the form tag, and set it equal to 
the name of the form group.

To associate each input element with the appropriate `FormControl` of the `FormGroup`, we set `formControlName` equal to the name of the right `FormControl`
of the `FormGroup`.

To extract the model from the `FormGroup`, we access its `.value` property. In this example, we displaying the model's JSON on the screen. As we type and modify
values in those input elements, we will automatically see the value reflected on the model.

## Next Steps

Our natural next step would be to validate the input fields. To pull validation off with template-driven forms is quite a messy task. Reactive Forms has a more
organized and easier way of tackling validation.