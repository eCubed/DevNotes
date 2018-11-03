# Angular Material and Reactive Forms - Revisit Partial Forms

A while back, we created a form that collected basic information about a user, including an object that contains address information. While everything works,
we will need to make some adjustments to it to prepare for loading an object into the form from the top-down, as opposed to just taking in the default or
empty values as done by the instantiation of the top-level object.

## Fixing the Partial Form to be a Form Creator

So far, I think we have discovered that the partial reactive form should be responsible for both determining its own local validations and for creating a
default `FormGroup`. This way, whatever parent reactive form component that will plop this child component in its markup doesn't need to know beforehand
the partial form details of the child, and can call upon the child (via static function) to create a new instance of the partial `FormGroup`.

The address partial form right now declares the `FormGroup` in its `ngOnInit()` function because it assumes that a phantom or arbitrary form has been
passed to it via the `@Input`, and that the `@ngOnInit()` should be responsible for patching such a form with the right form descriptors.

```javascript
ngOnInit() {
    /* Since the address form has already been created (though supposedly not
        declared from the parent component), we will fill it here.
    */
    if (this.addressForm != null) {
        this.addressForm.addControl(
            'address1',
            new FormControl('', Validators.required)
        );
        this.addressForm.addControl(
            'address2',
            new FormControl('')
        );
        this.addressForm.addControl(
            'city',
            new FormControl('', Validators.required)
        );
        this.addressForm.addControl(
            'region',
            new FormControl('', Validators.required)
        );
        this.addressForm.addControl(
            'postalCode',
            new FormControl('', Validators.required)
        );
    }
}

```

While we're at it, let's also rename `@Input() addressForm` to just `@Input address`. When working with reactive forms, we know that everything
passed between components is a `FormGroup`, so we don't need to append `-Form` to the names.

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

  constructor() { }

  ngOnInit() {    
  }

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
}
```

Now, at the parent component, let's use `createAddress()` instead of `new FormGroup({})` when instantiating the top-level `FormGroup`. Also there,
while we're at it, let's remove the `-Form` suffix from the top-level `FormGroup` object.

```javascript
import { Component, OnInit } from '@angular/core';

import { FormGroup, FormControl, Validators } from '@angular/forms';

import { AddressFormComponent } from '../../components/address-form/address-form.component';

@Component({
  selector: 'app-registration-composite',
  templateUrl: './registration-composite.component.html',
  styleUrls: ['./registration-composite.component.scss']
})
export class RegistrationCompositeComponent implements OnInit {

  registration: FormGroup;

  constructor() { }

  ngOnInit() {
    this.registration = new FormGroup({
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
}
```

After all the modifications made, the form still works the same exact way as before. We do this to prepare for the situation where we would load an
object to the form from the top-level.

## Preloading an Object onto the Top-Level Reactive Form

Back when we worked with template-driven forms with nested components, all we needed to do was pass complex properties as `@Input` to those sub-components,
and so on until we reach the child-most component. We automatically saw the components and subcomponents fill up. Unfortunately, filling a component with
complex objects isn't as easy when working with reactive forms. While we can bind objects and their properties to the UI in our markup with template-driven forms, 
we actually associate basic input elements with a `FormControl` and we pass `FormGroup`s as `@Input` to partial reactive forms. Since reactive form
objects (`FormGroup`, `FormArray`, `FormControl`, etc.) are not our POJOs, be forewarned:

**Filling a reactive form with object property values will require manual coding.**

We initialized the form by manually building the `FormGroup` object, filling it with default initial values for each `FormControl`. We will have to
also manually build the top-level `FormGroup` in somewhat of a similar fashion for "edit mode", that is, when we want to fill the form with values from
a POJO we may have received from an API call, but we will need to manually pick out properties and sub-properties of that POJO to initialize each `FormControl`.
This sounds quite tedious and we will need to do this in every complex form. This is a fairly huge price to pay in terms of development time and effort,
but it makes validation so much easier, which is a far more difficult feat to pull with template-driven forms.
...



