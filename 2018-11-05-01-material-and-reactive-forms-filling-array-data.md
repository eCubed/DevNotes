# Angular Material and Reactive Forms - Filling Array Data

We have previously seen a form with a partial reactive form load data from a POCO with `FormGroup.patchValue({})`. What if the top-level `FormGroup` has a property intended
to be an array of data, thus, a `FormArray`? We have seen in our `ToDoEditor` how to declare a `FormArray` and allow the user to add, delete, and reorder its items at runtime.
We did not yet explore how to pre-load an object into a `FormGroup` that contains a `FormArray`.

## Pre-loading Data Into a FormGroup with a FormArray

Let's go back to our `ToDoEditor` component's `ngOnInit()` function and let's try to load an object with an array of items.

```javascript
...
export class ToDoEditorComponent implements OnInit {

  toDo: FormGroup;

  constructor() { }

  ngOnInit() {
    this.toDo = new FormGroup({
      name: new FormControl(''),
      items: new FormArray([], Validators.minLength(2))
    });

    this.toDo.patchValue({
      name: 'My Workout',
      items: [
        { name: 'pushups', done: false },
        { name: 'situps', done: true }
      ]
    });
  }
  ...
}
```

On the top-level `FormGroup`, `toDo`, we specified `item` which is to be a `FormArray`. Since `item` is *only* declared to be a `FormArray`, we initialize it with an empty
array. Now, let's remind ourselves that the first argument of `FormArray` is *not* an array of POJOS, but an array of `AbstractControl`s, which could be `FormGroups`, `FormControls`,
or even another `FormArray`.

Now, we attempt to pre-load the form with data from a POJO. That POJO has an `items` property, which is an array *of POJOs*. So far, we are good because we specify `items` in our
main POJO, and there also is a corresponding `items` `FormArray` in the top-level form we are filling.

When we run the app, the value `name` makes it to the UI, but the `items` in the POJO don't make it into the form. Why? Furthermore, why did it work when we preloaded a complex
property when we loaded the address information onto the UI using the same pattern? What's the difference?

Let's recall the filling code:

```javascript
export class RegistrationCompositeComponent implements OnInit {

  registration: FormGroup;

  constructor() { }

  ngOnInit() {

    // This code is to set up the form for create mode!!!
    this.registration = new FormGroup({
      ...
      email: new FormControl('', Validators.required),
      address: AddressFormComponent.createAddress('','','','','')
    });

    // This code is to set up an object
    let regInfo = {
      ...
      email: 'billsmith@gmail.com',
      address: {
        address1: '15 Elmore Av',
        city: 'Davenport',
        region: 'IA',
        postalCode: '52807',
      }
    }

    // Now, how do we fill that reg form with the data on the POJO?

    this.registration.patchValue(regInfo);
  }
```

When we instantiated `registration`, which is a `FormGroup`, the `address` was also initialized to be a `FormComponent`. Recall that `AddressFormComponent.createAddress('','','','','')`
returns a `FormGroup` that already contains `FormControls` for each of `address`'s properties. Now, when `patchValue` takes in `regInfo`, which we also filled `address` with
an object, the form *already* has the `FormControls` whose names match the properties of `address`, and `patchValue` was able to load the property values into the appropriate
controls.

In our `items` `FormArray` declaration, `items: new FormArray([], Validators.minLength(2))` only says that `items` is a `FormArray`, but it does *NOT* have information
about what `FormControl`s might be inside a to-do item `FormGroup`! Though `items` was included in the POJO, there were no `FormControl`s whose name matches `name` and `done`
of a to-do item to match and then eventually fill. This is why `items` did *not* make it into the UI.

So what is the *cure* for this `FormsArray` problem when loading an array of data into a form? We obviously don't want to load a `FormGroup` to the `FormArray` at instantiation,
because that would automatically load unwanted data in create mode.

Let us momentarily fix the `ToDoItemComponent`'s `CreateToDoItem()` to also take in the `done` value:

```javascript
export class ToDoItemComponent implements OnInit {
  ...

  public static createToDoItem(name: string, done: boolean) : FormGroup {
    return new FormGroup({
      name: new FormControl(name, Validators.required),
      done: new FormControl(done)
    });
  }
}
```

Henceforth, every partial reactive form's static `create` function should have parameters for every `FormControl` that we want to show on the screen. Alternatively, it could have a
single parameter that exactly is the POJO itself. But for now, since our object has relatively few properties, we can have a parameter per property.

Unfortunately, the fix for filling a `FormArray` from a POJO's array of items is quite a tedious and involved process, at least for now. The following is the new `ngOnInit()` of the
`ToDoEditorComponent`. We will discuss the changes to the code shortly.

```javascript
export class ToDoEditorComponent implements OnInit {

  toDo: FormGroup;

  constructor() { }

  ngOnInit() {
    this.toDo = new FormGroup({
      name: new FormControl(''),
      items: new FormArray([], Validators.minLength(2))
    });

    let toDoObject = {
      name: 'My Workout',
      items: [
        { name: 'pushups', done: false },
        { name: 'situps', done: true }
      ]
    };

    this.toDo.patchValue(toDoObject);

    let toDoItemForms = [];

    for(let i = 0; i < toDoObject.items.length; i++) {
        toDoItemForms.push(ToDoItemComponent.createToDoItem(toDoObject.items[i].name, toDoObject.items[i].done))
    }

    this.toDo.setControl('items', new FormArray(toDoItemForms, Validators.minLength(2)));
  }

  ...
}
```

As usual, we declare the top-level `FormGroup`, we get a hold of the POJO (`toDoObject` in this case) with all of the data intact including arrays. We then call `FormGroup.patchValue()`.
This will automatically fill all `FormControls` with the expected data, except arrays.

The next thing we'll need to do is instantiate an array of forms: `let toDoItemForms = []`. Now, we *manually* iterate `toDoObject.items`, and build a `FormGroup` per item (with
`ToDoItemComponent.createToDoItem(...)`), and then push that `FormGroup` into the `toDoItemForms` array. Now, we use the top-level `FormGroup`'s `setControl` function to
actually create a new instance of a `FormArray`.

One might notice why we specify an empty `items` `FormArray` in the initial top-level `FormGroup` instantiation if we would overwrite it anyway with `FormGroup.setControl(...)`.
The reason for this is this very form is exactly the same one we would use when creating a new record or editing something that already exists. Note that the code to fill the
top-level `FormGroup` would usually be put in a `.subscribe()` callback function, including explicitly filling the `FormArray`.

Note that at this time, there is no better way to fill a `FormArray` than explicitly building each `FormGroup` off each item, and then adding it to the `FormArray`.




