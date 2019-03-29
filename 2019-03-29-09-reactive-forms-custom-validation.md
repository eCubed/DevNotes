# Reactive Forms - Custom Validators

So far, we have used a couple of validators that came with `@angular/forms`, specifically `required`, `minLength`, and `maxLength`. There are other validators
like `pattern` to match a regular expression, and a convenient `email` validator to make sure that the email inputted is in a correct format. While those validators
are commonly used, our app can get complicated enough that it would require some validations that the out-of-the-box ones cannot perform. In this article, we will
explore how to create our own validators that would do exactly what we need.

## Anatomy of a Validator

A validator, in its simplest form, is a function that takes in an `AbstractControl`, and returns a special object if validation is supposed to fail, and null
when the validation passes.

```javascript

function genericBasicValidator(control: AbstractControl) : {[key: string]: any} | null {
    /* Access the value of the control: control.value.
       Examine it for anything wrong.
    */
    ...
    let somethingWrong: boolean = ...;
    let anyAppropriateValue: any = ...;

    return (somethingWrong) ? { 'validatorKey': anyAppropriateValue } : null;
}
```

The `AbstractControl` has a `.value` field, which the validator will examine and run through its own validity tests. Typically, the value is a string if the
validator were applied to a text box or text area, but it can be a `FormArray`, so we need to keep in mind what we're validating when we write our validators.

The return type is really a simple JSON object, but the key *has* to be a string. The validator key that we specify in this object is important because the framework
will add it to a `FormControl`'s, `FormGroup`'s, or `FormArray`'s errors object, and we would need to reference the validator key in the markup to know whether
to show the corresponding error message of the validator that failed.

What if we wanted a validator to have parameters? Do we simply add parameters to the function above? No. The framework specifically asks for a function with the
exact function signature as the function above, that is, same parameter types, and same return type. The trick is we will need to create an outer function that
takes in the necessary parameters, and then *return a function* whose signature *is* the same as the function above.

```javascript

function validatorWithParameters(param1: any, param2: any ...) {
    
    return (control: AbstractControl) : {[key: string]: any} | null => {
        /* Access the value of the control: control.value.
           Examine it for anything wrong, using the parameter values of the outer function as appropriate.
        */
        ...
        let somethingWrong: boolean = ...;
        let anyAppropriateValue: any = ...;

        return (somethingWrong) ? { 'validatorKey': anyAppropriateValue } : null;
    }
}

```

We can put these validators among other validators in a form descriptor. For example:

```javascript
this.formGroup = new FormGroup({
    'name': new FormControl('', Validators.compose([
        Validators.required,
        genericBasicValidator,
        validatorWithParameters(param1, param2....)])
    ...
});
```

Notice that for a parameter-less validator, we merely list the function's name. However, when a validator has parameters, we actually call the validator function
and pass in the necessary parameters, and that outer validation function will return a function that is perfectly fit to be a validator.

## Creating Validators

We'll create a validator, albeit, something silly, that illustrates how to construct one. Though possible with the built-in `pattern` validator, we'll still create
it. Let's create a string validator that cannot contain an occurence of the word "retard".

```javascript
function cannotContainRetardValidator(control: AbstractControl): {[key: string]: any} | null {

    return (control.value.includes("retard")) ? { 'containsRetard': true } : null;
}
```

We can now then add this to a `FormControl`'s validation array:

```javascript
this.formGroup = new FormGroup({
    'name': new FormControl('', Validators.compose([
        Validators.required,
        cannotContainRetardValidator
     ]);
    ...
});
```

But that validator is too specific. How about we generalize it a bit so we can pass in any word that isn't allowed? We'll create a function that returns the validator
function that Reactive Forms requires:

```javascript
function cannotContainWordValidator(disallowedWord: string) {
    return (control: AbstractControl): {[string: key]: any } | null {
       return (control.value.contains(disallowedWord)) ? { 'containsDisallowedWord': true } : null;
    }
}
```

Now, we can add this to a `FormControl`'s validation array:

```javascript
this.formGroup = new FormGroup({
    'name': new FormControl('', Validators.compose([
        Validators.required,
        cannotContainWordValidator('idiot')
     ]);
    ...
});
```

What if we wanted a validator that cannot contain any words in a given array of strings? We'll leave that as an exercise!

Notice that we *just* put the value `true` as a value of the validator key. Indeed, it can be any object, but we don't need to complicate it beyond necessary.
After all, we'll only test for the *mere existence* of a property named after the validator key in the `errors` object in the markup!

Where do we write our validators? We certainly can write those functions right on the component that hosts the reactive form. Typically, custom validators are
component-specific that it's just fitting to put them in component code. Sometimes, we may put validators in a separate .ts file, especially if it's a validator
that many forms can apply.

## Next Steps

There is one major kind of validator that we have yet to cover, and those are the async validators - typically validators that would require to call an external
API to determine whether the inputted data is legit. We usually would like to validate a username or email address on a registration form to make sure it's not
yet taken, but an async validation can be more complicated than just validating string values against a database. The important thing is we are able to create
validators that call APIs to determine whether an input value is valid or invalid.