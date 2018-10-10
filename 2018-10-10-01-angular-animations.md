# Angular Animations

It is true that we can create animations on an Angular app using CSS3 transforms and transitions. However, this method will require us explicitly 
create specific logic that ultimately decides when to add and remove CSS classes to and from an element, and the addition or removal of these CSS
classes will trigger the animation. There is an Angular way of doing this without explicitly adding and removing CSS classes. The Angular way of
implementing animations will also not have us CSS classes including transition/transform information. Instead, we declare transitions right at the
component using some Angular animation APIs.

Our approach to Angular animations will not be very thorough and detailed about CSS or Angular animations itself. We'll approach it as "get something
done right away".

# Importing Animation Modules

Before we can tackle animation, we must import two modules from Angular into our app.component.cs file:

```javascript
...
import { BrowserModule } from '@angular/platform-browser';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations'
...

@NgModule({
  imports: [
    ...
    BrowserModule,
    BrowserAnimationsModule
  ],
  ...
})
...
```

## Animations on Show and Hide of a Component

We know that we can show and hide components or elements using the `*ngIf` directive. It works, however, the showing or hiding of the component is
abrupt. There is no smooth transition, like, for example, fading in or out, or resizing. We will fix that problem.

First, we will create an **animation trigger**, which really is *the animation* that describe what the animation should look like, including how long
it would last, which physical properties should be animated, the values of each physical property at the start and at the end of the animation, and how
to progress through the animation over its duration (known as easing). Our first animation trigger would fade something in or out.

In a hosting component's .ts file, we will need to add the following imports:

```javascript
import {
  trigger,
  style,
  animate,
  transition,
  query
} from '@angular/animations';
```

Then, at the `@Component` class decoration, we will need to specify the `animations` array, which is an array of animation triggers. We load it
with our first animation trigger:

@Component({
  selector: 'my-component',
  ...
  animations: [
    trigger('opacityTrigger', [
      transition(':enter', [
        style({ opacity: 0}),
        animate(1000, style({opacity: 1}))
        ]
      ),
      transition(':leave', [
        style({ opacity: 1}),
        animate(1000, style({opacity: 0}))
        ]
      )
    ])
  ]
})

Let's briefly describe what's going on. We define a trigger using the `trigger()` function. The first parameter is the trigger name, which we'll
refer to later from markup code. The second parameter is an array, which is an array of transitions. In this example, there are two transitions,
`:enter` and `:leave`. `:enter` is what happens when the component or element is created (shown), and `:leave` is what happens when the 
component or element disappears (literally removed from the DOM). Both `:enter` and `:leave` are standards in Angular, so they must be spelled
correctly for Angular to play our intended animations for those two scenarios.

The second parameter of `transition()`, is an array, and for now, an array of arbitrary animation objects. We will not get into the details of
Angular animations, like what type each of these functions return, if any. For that array, we'll regard that the first item describes the physical
appearance of an element at the start of the anmation. The second item is how to animate it - including duration in ms, and then what the component
would look like at the end of the animation.

For our case, we have this  `opacityTrigger` that specifies two transitions - `:enter` for showing the component, and `:leave` for removing the
component from the web page. For the `:enter` transition, we'll start with a style that sets opacity to 0, AKA, don't actually show it *yet* when
the component is added to the DOM. Then, we'll give that component 1000 ms to smoothly transition between opacity 0 to 1 (fully visible). This smoothly
fades in the component. For the `:leave` transition, we do the oppoosite - start with opacity 1 (fully visible), and then fade to opacity 0 over
1000 ms. It is important to note that when the `:leave` transition is playing out, the component is *still* in the DOM, even upto the point in time
when the opacity is actually 0. It's only right after when the `:leave` transition is complete when the component is actually taken out.

To apply the animation, we use the `@triggerName` notation on the element or component that also has the `*ngIf` directive applied to it:

```html
<div @opacityTrigger class="show-hide-div" *ngIf="divShouldShow">
  This is the div I want to show and hide, but with smooth animated transitions when showing and hiding.    
</div>
```

The `*ngIf` would be responsible for when to show and hide the element depending on the hosting component's events. However, referring to the
animation trigger would apply the declared animations when the component appears or disappears.

Now, it is also possible to apply an animation trigger to an element or component where the `*ngFor` directive is applied:

```html
<ul>
  <li @opacityTrigger *ngFor="let item of stuff">
    {{ item }} <a (click)="removeItem(item)">Remove</a>
  </li>
</ul>
```

Notice that there isn't an `*ngIf` decorator that determines whether to show or hide the list item. Angular knows when each list item has been added
to the DOM, and will apply the @trigger's `:enter` transition automatically. Angular would also know when an item is removed (whose logic we will
not show since this should be basic Angular principle by now), and would automatically apply the @trigger's `:leave` transition automatically right
before the component is removed from the DOM (when logic in code removes the item from the array).

## Next Steps

We've briefly introduced Angular Animations so far, and demonstrated how and that they work. There are far more things to learn about Angular animations,
and we'll quickly describe them below:

**Animation States.** So far, we have only described two transitions - `:enter` and `:leave` which play out when a component is created and destroyed,
respectively. There are times that we would want to animate an element based on some logical states of our application, not just animate when it
shows up or disappears. For example, what if we wanted to animate between a selected vs. unselected physical state? In this case, the element or
component will always be present on the screen, so `:enter` and `:leave` won't really apply. We know that selected and unselected will probably have
different background colors, but we'll need to declare some animation info so that the background color would blend from one color to another smoothly.

**Animatable Properties.** We have shown that opacity is an animatable property. There are other CSS properties that we may apply smooth transitions
to. For now, any CSS property whose value is a rational number, or a color is animatable. This includes height, width, margins, padding, top, left, bottom,
right, border thickness and radii, any color property, opacity, font size, etc. There are also unanimatable CSS properties like position (relative,
fixed, absolute), font-family, box-sizing, and background-image since their values aren't even numeric. though z-index's value is a number, it's always
an integer, so it won't have a smooth transition at all. If anything, changing the z-index, even when attempted to animate, will abruptly show or hide
the component as soon as the z-index crosses the z-index of another element - not mouch animation there.

**Animating Route Changes.** To add a little bit of life to an application, we can animate the entry and exit of a routed component whenever the user
clicks on something that brings them to another route in the app. Yes, we still somewhat use Angular animations, but we will need to prepare a few
things in the upper levels of our entire application to get this to work.

**Animation on Dynamically Created Components.** This is a fairly advanced topic, though it is easy to assume that animation will work the same with
dynamically-created components as with anything that `*ngIf` or `*ngFor` will manipulate. We would now be programatically and explicitly be inserting
components on the fly, not putting them in markup which Angular would already have a hold of whether or not it starts out being physically present or
absent in/from the DOM at runtime. The `:leave` phase would be in trouble. Since we would be removing the component from outside the component, 
Angular somehow doesn't pick up the `:leave` transition that should happen right before component destruction. We will investigate this in a later
article!

