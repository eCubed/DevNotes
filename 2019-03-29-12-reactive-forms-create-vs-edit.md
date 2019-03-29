# Reactive Forms - Creating vs. Editing

So far, we have always created Reactive Form components, initializing it with default values. What would we need to do when we get a plain JS object maybe from
a `GET` api call for editing? In the easiest scenario, the POJO returned has no complex properties or arrays. We would simply create a `FormGroup` with a
corresponding `FormControl` for each of the POJO's properties, and manually initialize each `FormControl` with values from the POJO's corresponding
properties. We would then associate each `FormControl` with an actual input element in the markup with the attribute `formControlName`, and we are done.
However, it gets complicated once our POJO has complex-typed properties and arrays.

In this article, we will devise a general plan or approach that will let us have the user create and edit an entity, and we shall assume that this entity can
have complex-typed properties and arrays.

(This article isn't complete, so I'll just write to get points across with minimum listing code or markup, as I'll provide those later).

## The Top-Level Form Component

Of course, we will know the JSON of the entity we'll want to create, otherwise it would be impossible to build the form to collect and to modify the values.
We know that there are two tasks - creating a new entity and editing an existing entity (typically pulled from an `GET` api call). There will need to be two
routes - one for creation and one for updating. The creation route will only make an api call to create the entity. The edit route will have 2 api calls - one
to obtain it by a unique id, and another one to update the entity. Since both routes manage the same JSON data structure, it would be too cumbersome to maintain
two copies of the top-level `FormGroup` with all the validations. It would also be cumbersome to create identical HTML markup, one for each route.

Instead of maintaining two `FormGroups` and two HTML markups, we will create a single top-level form component that both create and update routes will host.
This top-level form should not care whether the intent is to create a new entity or update an existing one. This form is concerned *only* with associating each
property with the appropriate Reactive Form controls, and more importantly, ensuring that the data entered is valid, as performed by its internal validation.

The top-level form component should *always* have a public static create function that takes in as parameters values for each property we would like the user to
modify. The values of these parameters become the initial value of the form for the update scenario. We could, and it is highly-recommended, that we also create
another public static createDefault function, which takes in no parameters that would be used by the create scenario. Otherwise, we would need to call up the
plain createFunction and manually pass in empty strings, 0, etc. - default values. We will get into the details of how to consistently write these public static
create functions.

So far, the top-level form is responsible for creating the proper `FormGroup`, for collecting the data from the user, and for performing validations.

## Create and Update Routes

Both create and update routes will store a reference to our top-level `FormGroup`. This way, we have the `FormGroup` ready for saving in either route.

As we may have done with template-driven forms, the update route takes in a route parameter, typically the unique id of the entity we want to edit. From this
parameter, we will need to make the API call, receive the POJO, and then nit-pick it so we can and eventually call the top-level form component's public static
create function that takes in parameters. For example, very roughly, this is what we would code in an update route:

```javascript
ngOnInit() {
  ...
  this.myService.getEntity(idFromRouteParam).subscribe(data => {
     this.myEntity = EntityFormComponent.createEntity(data.id, data.property1,... data.propertyN);  
  });
  ...
}
```

On the other hand, the create route will only need to create a default component without calling an API:

```javascrit {
ngOnInit() {
  ... 
     this.myEntity = EntityFormComponent.createDefaultEntity();  
  });
  ...
}

Now, on the markup for both routes, we see the same thing:

```html
...
<app-top-level-form [entity]="myEntity"></app-top-level-form>
...
```

When it comes to saving, though, they are different again. Both would have save buttons, but the create route will ultimately make a POST request and the
update route will make a PUT request. Now, for the create route, depending on the app's need, may want the user to immediately get out of the route and back to
the listing page, forward the user to the corresponding update page, or remain on the create page. For the third case, we will need to write a mechanism to
detect whether a save button click will issue a POST or PUT request. This discussion will for managing the create route calls for its own separate article.

# Managing Complex and Array Properties

It would be extremely cumbersone to need to manually build the entire `FormGroup` at the top-level form component, so we will need to define a plan for
handling complex properties and arrays. The key to streamlining this process is in the public static create functions of the top-level form component *and all
partial form components*.

Let us acknowledge that we have a complex POJO, with primitive, complex, and array properties.

First, let us explore the public static create functions that take in parameters:

```javascript
public static createEntity(id: number, primitiveProperty1: primitiveType,.., complexProperty1: any,..., arrayProperty1: any[],...) {
    /* First, take care of the complex properties - create the formGroup for each one
    */

    const complexPropertyFormGroup1 = ComplexProperty1FormComponent.create(complexProperty1.value1,... complexProperty1.valueN);
    ...
    const complexPropertyFormGroupN = ComplexPropertyNFormComponent.create(complexPropertyN.value1,... complexPropertyN.valueM);

    /* Then, take care of each array
    */

    const arrayProperty1FormArray = new FormArray([]);
    arrayProperty1.forEach(arrayProperty1Item => {
       arrayProperty1FormArray.push(
         ComplexObject1OfArrayProperty1FormComponent.create(arrayProperty1Item.value1.....)
       );
    });

    /* Finally, build the `FormGroup` and return it.
    */

    return new FormGroup({
       id: new FormControl(id),
       primitiveProperty1: new FormControlId(primitiveProperty1),
       ...
       complexProperty1: complexPropertyFormGroup1,
       ...
       arrayProperty1: arrayProperty1FormArray,
       ...
    });
}
```

The approach is similar with the createDefault function. One key difference is, depending on our app, we could prepopulate arrays with a single item, or leave
them blank. All of the nested complex properties should call their corresponding component's public static create default functions, though.

We will also use the same exact approach when creating the public static create functions for the partial reactive forms.

Note that the top-level create function is still quite involved, however, we will only need to initialize form controls for each immediate property, not
drill down to every possible descendant properties and manually create them at the top.