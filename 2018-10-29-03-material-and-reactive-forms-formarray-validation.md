# Angular Material and Reactive Forms - FormArray Validation

We have previously validated values of textboxes, both declaring validations on the corresponding `FormControl`, and both in the markup to show the
error message that corresponds to a validation rule. We have not yet validated against the array.

In this article, we'll add a validation rule that says there has to be at least two items in the to-do list. Let's begin by modifying the Typescript
code of the parent reactive form component:

```javascript

import { ..., Validators } from '@angular/forms';
...
export class ToDoEditorComponent implements OnInit {
   
  ...
  toDoForm: FormGroup;
  
  ngOnInit() {
    this.toDoForm = new FormGroup({
      name: new FormControl(''),
      items: new FormArray([], Validators.minLength(2))
    });
  }
  ...
}
```

We've seen the `.minLength` validator before. We used it to validate strings on textboxes which have a `length` property. A `FormArray` *also* has a `length`
property, so the `.minLength` (and also `.maxLength`) validators will work. Back at the markup, we add the error message logic.

```html
<mat-card>
    <mat-label>Items</mat-label>
    <app-to-do-item
        [toDoItem]="toDoItem"
        [index]="i"
        [canMoveUp]="i > 0"
        [canMoveDown]="i < toDoForm.controls['items'].controls.length - 1"
        (onRequestDelete)="onRequestDeleteHandler($event)"
        (onRequestMoveUp)="onRequestMoveUpHandler($event)"
        (onRequestMoveDown)="onRequestMoveDownHandler($event)"
        *ngFor="let toDoItem of toDoForm.controls['items'].controls, let i = index">
    </app-to-do-item>
    <mat-error *ngIf="toDoForm.controls['items'].invalid">
        <mat-error *ngIf="toDoForm.controls['items'].errors.minlength">
            There needs to be a minimum of two items!
        </mat-error>
    </mat-error>
</mat-card>
```
We also went ahead changed the div that used to wrap `app-to-do-item` and changed it to `mat-card` so that `mat-error` and `mat-label` will obtain the look of
Angular Material.

The way to markup the error messages is exactly the same as when we're validating a string value of a textbox.

## Next Steps

We will need to perform some `FormArray`-wide validation beyond number of items count, and beyond validation of each individual item. We're talking about the kind of validation
that needs to first collect data from *all* items, and performing validation on the collected data. For example, we'll consider the array valid if none of the items have the same
name. Conversely, if two or more items have the same name, the entire `FormArray` is invalid. We cannot perform this kind of validation at the array item level, because each item
doesn't (and shouldn't) know about the data in its siblings. What we're going to do next is to explore custom validation. We'll first start with creating custom validators for
string values, and then move on to custom validators against a `FormArray`.