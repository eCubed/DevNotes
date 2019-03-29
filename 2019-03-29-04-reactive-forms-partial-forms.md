# Reactive Forms - Partial Reactive Forms

We have previously discussed the situation when the model of our form has a property that is of an object type, and this property has many so many properties
of its own that it probably calls for its own component to layout a *partial* form for it. Otherwise, our main form would get too crowded. We have created partial forms
as components before when working with template-driven forms. The partial form component would have an `@Input` where we would pass in a complex property of
an object, then that component would be responsible for layout and binding.

We were previously concerned with how we're going to pull componentization when working with partial Reactive Forms because (1), the partial form cannot have
a form tag inside (to specify a top-level form group so we can pair up individual elements with `FormControl`s as we've done at top-level reactive form), and (2)
we want to ensure that the partial form's local validations will be "heard" by its parents to determine the validity of the top-level object.

## Preparation

We will pick up where we left off with our easy registration form. It only has 3 immediate fields - first name, last name, and email. We have also applied validation
on those fields. We shall regard the registration form as the parent or top-level form. So far, we have the top-level form:

```html
<form [formGroup]="registrationForm">
    <div>
        <div>
          <input type="text" formControlName="firstName" placeholder="First Name" />
          <div *ngIf="registrationForm.controls['firstName'].invalid">
            <div *ngIf="registrationForm.controls['firstName'].errors.required">
                First name is required.
            </div>
            <div *ngIf="registrationForm.controls['firstName'].errors.minlength">
                First name must be at least 3 characters long
            </div>
            <div *ngIf="registrationForm.controls['firstName'].errors.maxlength">
                First name cannot exceed 20 characters
            </div>
          </mat-error>
        </div>
        <div>
          <input type="text" formControlName="lastName" placeholder="Last Name" />
          <div *ngIf="registrationForm.controls['lastName'].invalid">
            <div *ngIf="registrationForm.controls['lastName'].errors.required">
                Last name is required.
            </div>
          </div>
        </div>
        <div>
          <input type="text" formControlName="email" placeholder="Email" />
          <div *ngIf="registrationForm.controls['email'].invalid">
            <div *ngIf="registrationForm.controls['email'].errors.required">
                Email is required.
            </div>
          </div>
        </div>
        <button type="submit" [disabled]="registrationForm.invalid">Submit</button>
    </div>   
    <div>
      <pre>{{ registrationForm.value | json }}</pre>
    </div>
</form>
```

And its component code:

```javascript
import { Component, OnInit } from '@angular/core';

import { FormGroup, FormControl, Validators } from '@angular/forms';

@Component({
  selector: 'app-registration-composite',
  templateUrl: './registration-composite.component.html',
  styleUrls: ['./registration-composite.component.scss']
})
export class RegistrationComponent implements OnInit {

  registrationForm: FormGroup;

  constructor() { }

  ngOnInit() {
    this.registrationForm = new FormGroup({
      firstName: new FormControl('', Validators.compose([
        Validators.required,
        Validators.minLength(3),
        Validators.maxLength(20)
      ])),
      lastName: new FormControl('', Validators.required),
      email: new FormControl('', Validators.required)
    });
  }
}
```

We know that we will need to put the partial form's tag somewhere in the top-level form's markup, but what do we pass to the partial form? On the Typescript code,
how do we specify in that top-level `FormGroup` that we intend to have a property that is a complex object? Wouldn't the top-level `FormGroup` need to know at
`ngOnInit()` the properties of that complex object? We will address these concerns as we move on. First, let us create the partial form.

## Partial Reactive Form Markup

We will now create a component that we will regard as a partial reactive form. It will contain the UI for the properties of an address - addres1, addres2, city,
region, and postal code. The way we construct a partial reactive form is nearly identical to how we would build a top-level reactive form. Here is the markup:

```html
<div [formGroup]="address">
    <h3>Address</h3>
    <div>
        <input type="text" formControlName="address1" placeholder="Address 1" />
        <div *ngIf="addressForm.controls['address1'].invalid">
            <div *ngIf="addressForm.controls['address1'].errors.required">
                Address is required
            </div>
        </div>
    </div>
    <div>
        <input type="text" formControlName="address2" placeholder="Address 2"/>
        <div *ngIf="addressForm.controls['address2'].invalid">
            <div *ngIf="addressForm.controls['address2'].errors.required">
                Address is required
            </div>
        </div>
    </div>
    <div>
        <input type="text" formControlName="city" placeholder="City" />
        <div *ngIf="addressForm.controls['city'].invalid">
            <div *ngIf="addressForm.controls['city'].errors.required">
                City is required
            </div>
        </div>
    </div>
    <div>
        <input type="text" formControlName="region" placeholder="Region" />
        <div *ngIf="addressForm.controls['region'].invalid">
            <div *ngIf="addressForm.controls['region'].errors.required">
                Region is required
            </div>
        </div>
    </div>
    <div>
        <input type="text" formControlName="postalCode" placeholder="Postal Code" />
        <div *ngIf="addressForm.controls['postalCode'].invalid">
            <div *ngIf="addressForm.controls['postalCode'].errors.required">
                Postal code is required
            </div>
        </div>
    </div>
</div>
```

We went ahead and put the validation markup. One thing that's different here from a top-level reactive form, is that we DO NOT have a form element at the root of this
component. We were concerned about how we would associate the whole markup with some local root-level form container since we coudn't have a form element. Instead, we 
have a root div element, to which we apply the `[formGroup]` directive to declare that this partial form is associated with `addressForm`, the very `FormGroup` that 
describes the model of our address. Each `formControlName` directive declared in each input is declared exactly the same way as it would be at a top-level reactive form.

# Partial Reactive Form Code

Here is the Typescript for the address partial reactive form:

```javascript
import { Component, OnInit, Input } from '@angular/core';

import { FormGroup, FormControl, Validators } from '@angular/forms';

@Component({
  selector: 'app-address-form',
  templateUrl: './address-form.component.html',
  styleUrls: ['./address-form.component.scss']
})
export class AddressFormComponent implements OnInit {

  @Input() address: FormGroup;

  public static createAddress(
    address1: string,
    address2: string,
    city: string,
    region: string,
    postalCode: string
  ) : FormGroup {
      return new FormGroup({
        'address1': new FormControl(address1, Validators.required),
        'address2': new FormControl(address2),
        'city': new FormControl(city, Validators.required),
        'region': new FormControl(region, Validators.required),
        'postalCode': new FormControl(postalCode, Validators.required)        
      });
  }

  constructor() { }

  ngOnInit() {
  }
}
```

The biggest key of partial reactive forms is that we take in an `@Input` that is of type `FormGroup`. With template-driven components, we are used to taking in
an `@Input` that is of the type of our model, but that is not the way of reactive forms. Let us be reminded that when working with reactive forms, we deal with
`FormGroup`s and `FormControl`s, not POJOs. This means that we will be passing around `FormGroup` and `FormControl`s between parent and children. Now, because
we have an `@Input` that is of type `FormGroup`, we *will actially* be passing in a `FormGroup` to this partial component from the top-level reactive form.

Now, from inside this partial reactive form component, we do NOT initialize the `FormGroup` as we do in a top-level reactive form, because it was passed down to us
from the parent component. Instead, our partial reactive form controller will have a public static function whose individual parameters are the data we want to take
in and pre-fill our form. Inside the public static function we return a `FormGroup`, and we have declared it as we always have before.

The public static function strategy is useful because it keeps the validation rules together with its form component. If ever we need an address `FormGroup` in MANY
parent forms, we can instantiate one from this single public static function.

## Reworking the Parent Reactive Form for Composite Data

The bulk of the work is performed in creating the partial reactive form component. Now, we need to put its markup tag in the top-level reactive form's markup.
But before we do that, we need to modify the top-level `FormGroup` instantiation to recognize the address partial form:

```javascript
ngOnInit() {
this.registrationForm = new FormGroup({
    firstName: new FormControl('', Validators.compose([
        Validators.required,
        Validators.minLength(3),
        Validators.maxLength(20)
    ])),
    lastName: new FormControl('', Validators.required),
    email: new FormControl('', Validators.required),
    address: AddressFormComponent.createAddress('','','','','')
});
}
```

We add `address` as a property, and we declare it to be an instance of a an address `FormGroup`, created off partial reactive form's public static function
`AddressFormComponent.createAddress(...). Notice that we pass in empty string values since a registration by nature, always starts out blank. However, we are establishing
a good practice here, as we'll later see for an "edit" scenario when we'll obtain objects from an API form.


Now, let us add the address component to the parent form's markup:

```html
<form [formGroup]="registrationForm">
  <div>
    <div>
        <div>
          <input type="text" formControlName="firstName" placeholder="First Name" />
          <div *ngIf="registrationForm.controls['firstName'].invalid">
            <div *ngIf="registrationForm.controls['firstName'].errors.required">
                First name is required.
            </div>
            <div *ngIf="registrationForm.controls['firstName'].errors.minlength">
                First name must be at least 3 characters long
            </div>
            <div *ngIf="registrationForm.controls['firstName'].errors.maxlength">
                First name cannot exceed 20 characters
            </div>
          </div>
        </div>
        <div>
          <input type="text" formControlName="lastName" placeholder="Last Name" />
        </div>
        <div>
          <input type="text" formControlName="email" placeholder="Email" />
        </div>
        <app-address-form [address]="registrationForm.controls['address']">
        </app-address-form>

        <button mat-raised-button color="primary" type="submit" [disabled]="registrationForm.invalid">Submit</button>
    </div>
    <div>
      <pre>{{ registrationForm.value | json }}</pre>
    </div>
  </div>
</form>

```
The address form component, if we recall, has the `@Input` of type `FormGroup` named `address`, so we pass into it the `address` `FormGroup` of the
top-level `FormGroup` that we have declared in `ngOnInit()`.

If we run this application now, we will see that we can still extract our model out of the top-level `FormGroup`, and the model, indeed, includes the complex
property we're expecting that holds address information. Since we're creating the `FormGroup` for the address form at the top level, albeit empty, the top-level
*does* consider the validations of the partial form - the entire form won't validate if there is a validation violation inside the partial form!

## Next Steps
Now that we have achieved partial reactive forms, and most importantly, recognizing their validations and violations at the top level, we are now ready to work
with arrays of complex objects. We will need to address how to represent an array of data when building `FormGroup`s, and how to properly markup the array data.
