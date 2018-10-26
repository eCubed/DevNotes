# Angular Material and Reactive Forms - Intro to Validation

Now that we have set up a basic reactive form, we are now ready for the next step - validation.

We are continuing to develop our basic registration form, and we'll state the following rules to start - each of those three fields are required. That is, all of them
cannot be null, and must have a minimum length of 1 character.

We'll refresh ourselves with the `FormGroup` declaration:

```javascript
this.registrationForm = new FormGroup({
    firstName: new FormControl(''),
    lastName: new FormControl(''),
    email: new FormControl('')
});
```

## Adding Validation

There are a few validations that come out of the box with Reactive Forms, and the required validation is one of them. We add a required validator as the second
parameter of the `FormControl`'s constructor:

```javascript
import { FormGroup, FormControl, Validators } from '@angular/forms';

this.registrationForm = new FormGroup({
    firstName: new FormControl('', Validators.required),
    lastName: new FormControl('', Validators.required),
    email: new FormControl('', Validators.required)
});
```

If we run the app right now, we won't immediately see the effect upon page load. If we touch one of the elements and then focus out, its graphics will turn red.
If we type anything in and then focus out, it will turn back to its normal colors.

And at this point, we have NOT declared any CSS. Material components know about invalid data, and knows how to alter the appearance to reflect invalid state.

## Corresponding Error Messages To The User