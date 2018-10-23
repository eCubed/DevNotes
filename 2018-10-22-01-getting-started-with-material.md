# Getting Started with Angular Material

I've touched Angular Material before but was somewhat confused. Also, I challenged myself with developing my own widgets from scratch, which is taking way
too much time. This is now a reboot into it, now, starting with Angular 6.

I went ahead and created a new angular app with `ng new my-app --style=scss --routing` because I want scss and I want routing even for this demo app.

## Preparation Before Setting Up Angular Material

Install the following: `npm i --save @angular/material @angular/cdk @angular/animations @angular/flex-layout`

Import `BrowserAnimationsModule and FlexLayoutModule` from inside `app.module.ts` and add them to the `imports` array:

```javascript
import { BrowserModule } from '@angular/platform-browser';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { FlexLayoutModule } from '@angular/flex-layout';
import { NgModule } from '@angular/core';

@NgModule({
  declarations: [
  ],
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    FlexLayoutModule,
    AppRoutingModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Go to `styles.scss` and import one of the built-in Angular Material themes. For this article, we chose the deep purple amber theme:

`@import "~@angular/material/prebuilt-themes/deeppurple-amber.css";`

We're going to want to use Material icons, but we don't want to install the whole font to our project because it will significantly increase the size
of the deployed app. Instead, we'll link to one out there online, from `index.html`:

```html
<head>
  ...
  <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
</head>
```

## Creating and Applying the Angular Material Module
So that we won't crowd `app.module.ts`, we are going to create a module just for Angular Material. I want the module to go under `/src/app/modules/`,
so I created the `modules` folder underneath `/src/app/`, then I cd'd to `/src/app/modules/`. From there, I ran

`ng g module Material --spec false`

The `--spec` flag with the value of `false` means that the CLI should NOT create the test file.

The CLI actually creates a folder underneath `/src/app/modules/` named `material`, in all lower-case. Inside that folder is the file `material.module.ts`
which we'll edit later on.

We then go back to `app.module.ts` and import our Material module, and register it in the `imports` array:

```javascript
import { BrowserModule } from '@angular/platform-browser';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { NgModule } from '@angular/core';
import { MaterialModule } from './modules/material/material.module';

@NgModule({
  declarations: [
  ],
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    MaterialModule,
    AppRoutingModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

## Modifying the Angular Material Module

So far, we have a default Angular Material Module, the file at `/src/app/modules/material/material.module.ts`. One important thing to note with
Angular Material development, is that we will need to nit-pick *exactly* which Angular Material modules our app will need. Yes, Material is subdivided
into multiple modules, a module for each component or widget. This means we pick out specific Angular Material modules to import and then register them to
both the `imports` and `exports` arrays of `MaterialModule` so that they become available throughout our app.

To see which components and widgets are available, we need to go to the [official documentation](https://material.angular.io/components/categories), then
choose a control or widget first (left-hand-side menu), then select the `API` tab for the control or widget we selected. The `import` statement will be
listed first, which contains the name of the module we need to import.

We will need to refer back to this documentation to know how to properly markup Angular Material components - it's not exactly straight-forward or obvious.
We'll see some html tags prefixed with `mat-` which we know are from Material. There are also directives we'll need to pay close attention to. We will
nest multiple levels of `mat-` tags to achieve what we want - either for layout or to even put one component down on the page. It's fairly complicated,
so at first, it'll feel like we'll merely plop down markup just to get something to work and/or display as we expect. Some of these components and widgets
actually have some programming APIs, like the dialog, for example.

We will not go through each Material component here, even some of the most common ones, since they're quite intricate and will need thorough explanations.
For now, we need to know that we pick out component and widget Angular Material modules, import them, and then register them to the `imports` and `exports`
array of our `MaterialModule`, and once done, we can plop down their corresponding markups in component code.

## Next Steps

Let's start learning some of the basics of Angular Material. We will using Material for the basic input controls instead of the plain html input controls. We will
later learn how to use some of the more sophisticated widgets and leverage their APIs in TypeScript.

