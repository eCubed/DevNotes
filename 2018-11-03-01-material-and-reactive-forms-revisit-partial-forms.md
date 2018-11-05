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

Suppose we received an object (POJO) from an API call. We have already created the form group as we have above, and we now have to fill it from the top
level.

```javascript
  ngOnInit() {

    // This code is to set up the form for create mode!!!
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

    // This code is to set up an object
    let regInfo = {
      firstName: 'Bill',
      lastName: 'Smith',
      email: 'billsmith@gmail.com',
      address: {
        address1: '15 Elmore Av',
        city: 'Davenport',
        region: 'IA',
        postalCode: '52807'
      }
    }

    // Now, how do we fill that reg form with the data on the POJO?

    this.registration.patchValue(regInfo);
  }
```
We first build an object (since this is just a demo, but the data would really come from an API GET response), `regInfo`. We then get a hold of the
top-level `FormGroup`, `registration`, and call upon the `patchValue()` function and pass to it our POJO, `regInfo`. This, in effect, will automatically 
fill the values of the form elements. Notice that the property names of our POJO match the `FormControl` names that we specified when we initially built the
`FormGroup`. This is crucial so that Reactive Forms knows which values to put in the right input elements. The complex property of `regInfo`, `address`,
will be reflected in the UI because we specifed `address` in the `FormGroup` with `addres: AddressFormComponent.CreateAddress('','','','','')`.
Had we not instantiated a `FormGroup` for `address`, we wouldn't get in the UI the values the properties of `regInfo.address`.

There is another function similar to `patchValue()` called `setValue()` that also takes in a POJO. `patchValue` is said to be "more lenient" because
it allows us to specify an incomplete set of properties of the POJO. For instance, our `regInfo`'s `address` object didn't specify `address2`. Passing
`regInfo` to `patchValue()` works because `patchValue()` will only modify the values we specify. Passing `regInfo` into `setValue()`, however, will
result in an error because it requires that the POJO specifies properties for every single `FormControl` declared in the original `FormGroup`. We will
favor `patchValue()` because the API may return an object that does not specify a property if that property's value is null, false, or a default value.

## Next Steps

We will now try to load a top-level object with a property that is an array.





