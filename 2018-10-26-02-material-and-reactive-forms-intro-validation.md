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

## User Feedback on Invalid Data

The only indication of invalid data we get so far is the input graphics turning red. We would like to get a message underneath each input element for a violation
made by the user, and then see it disappear if the data at the moment no longer violates it. Also, we will want that submit button disabled when there is ANY
violation at all.

Let's start with making a specific message show up underneath an input control whose value is invalid at the moment. We'll focus on the `firstName` `FormControl`.

```html
<mat-form-field>
    <input matInput type="text" formControlName="firstName" placeholder="First Name" />
    <mat-error *ngIf="registrationForm.controls['firstName'].invalid">
        <mat-error *ngIf="registrationForm.controls['firstName'].errors.required">
            First name is required.
        </mat-error>
    </mat-error>
</mat-form-field>
```

Angular Material comes with the `mat-error` component which serves as the container for error messages. Notice that we have two nested `mat-error` tags. The
outer `mat-error` will show up if the corresponding `FormControl`'s invalid state is true. Also, take note of the way we access a `FormControl` of a `FormGroup` -
via the `FormGroup`'s `controls` array. The inner `mat-error` will show for when a specific validator has been violated. We access the `errors` object of the
`FormControl`, and check whether the `required` validator in specific exists as one of its properties. Reactive Forms automatically fills and empties the `.errors`
object when the data is invalid or valid, respectively. Later, we will include more validation rules, and each additional validation rule would get its own
`mat-error` component with its own message.

We'll leave it as an exercise to apply the same required validation for the two other fields.

What about disabling that submit button when there are violations at all?

```html
<button mat-raised-button color="primary" type="submit" [disabled]="registrationForm.invalid">Submit</button>
```

The invalid state of the entire form (`FormGroup`) is derived from all its "children" and "descendant" `FormControl`s. Reactive Forms does that work for us, so
all we have to do is bind the `[disabled]` property to when `FormGroup.invalid` is true.

When we first load the app, we see no error messages or "errorly" graphic indications, however, the submit button is now disabled because none of the required values
are filled. As soon as the last required text box is filled, that submit button will become enabled. As soon as there becomes a validation, in our case, when the
user empties one of the text boxes, it becomes disabled again.

## Multiple Validators

In real life, being a required field is often not enough to consider the piece of data valid. Suppose we want to add a rule that will require the string to be
so long or short, like greater than 3 characters but less than 20, for example? How would we do that?

To accomodate our character count rules, `Validators` comes with validators `.minLength(number)` and `.maxLength(number)` along with `.required`. We
combine multiple validators together with `Validators.compose([])`, which takes in an array of validators. Note that the order in which we put validators does
NOT matter because all of them have pass in order for the entire `FormControl` to be considered valid.

```javascript
this.registrationForm = new FormGroup({
    firstName: new FormControl('', Validators.compose([
        Validators.required,
        Validators.minLength(3),
        Validators.maxLength(20)
    ])),
    lastName: new FormControl('', Validators.required),
    email: new FormControl('', Validators.required)
});
```

Now, we would want to display the corresponding error messages for having invalid string lengths:

```html
<mat-form-field>
    <input matInput type="text" formControlName="firstName" placeholder="First Name" />
    <mat-error *ngIf="registrationForm.controls['firstName'].invalid">
        <mat-error *ngIf="registrationForm.controls['firstName'].errors.required">
            First name is required.
        </mat-error>
        <mat-error *ngIf="registrationForm.controls['firstName'].errors.minlength">
            First name must be at least 3 characters long
        </mat-error>
        <mat-error *ngIf="registrationForm.controls['firstName'].errors.maxlength">
            First name cannot exceed 20 characters
        </mat-error>
    </mat-error>
</mat-form-field>
```

If we run the app at this point, when we type various strings of various lengths (including 0) into that `firstName` `FormControl`, we will see the error message
that corresponds to the current violation, if we broke a rule.

## Next Steps

There are other built-in validators that come shipped with the framework, and we'll leave those for independent exploration and as-needed bases. There is a `.pattern`
validator to require that a string matches a regular expression. There is also `.email`, which is a convenient `.pattern` validator to check proper email format.
It is also possible to create our own validators if the built-in ones don't suffice our needs. It is also possible to validate asynchronously, for example, consider
a username field invalid if an API call returns that it's already taken. We will not cover those more advanced validation topics as this article is just an introduction
to validation.