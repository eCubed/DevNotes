# Angular Reboot

By this time, I have accumulated a good amount of Angular knowledge, developed some front-end apps using even the latest version 6.x,
and even developed my own schemes and approaches on how to achieve certain goals. But I have not documented them. I will confess that I 
do not fully understand much of what I've developed and I don't know a lot of crucial responsibilities, for example, how to properly 
set up and conduct front-end tests. This undertaking will be another "fresh start" dive into Angular development, and this time, I'll 
take it more seriosly than merely just being able to deploy an app to production.

In this reboot, I finally want to fully understand and correctly implement and practice the following:

1. **SCSS.** I understand it. I know the syntax. I know how to import SCSS files into an SCSS file of one of my components. There are a
lot of things I'm not sure about. Is it alright to write two style blocks of the same signature (I mean accessor type (class, element,
id, etc) on different components? I thought that when SCSS gets compiled, the CSS will add a component-specific prefix to the signature
so that nothing would ever clash. I also heard that it is not advisable to write CSS classes in files that would be imported from
components' SCSS files, because merely importing them would create duplicate CSS class declarations in CSS, which would confuse the
system.
2. **Animation.** I have a few lines of animation code that I have copied from various sources. I've only ever used the fade-in and
fade-out, but could not get ones that involved physical movement of the elements to work correctly. Maybe animations are now easier and
require less code. And yes, those keyframe-involved ones that I've always ignored - I'll get at them now.
3. **CSS Positioning.** Yes, even after almost 20 years of CSS experience, I still don't have CSS positioning down pat, especially when
parent-child and relative-absolute positioning on either one of them is involved, and they're both buried deep in the DOM.
4. **RXJS.** Sure, I've used subscriptions before, know when to issue that my data changed, and know how to listen to when my data
changes. I'm still confused on why there are such things as Subjects, Behavior Subjects, Subscriptions, Observables, and many more,
and the difference between each one. I've "beat around the bush" with RJXS and "called it a day" when I used it just to be able to
obtain the response of an API call.
5. **Any Testing.** I understand unit testing when it comes to ensuring that the business logic is correct, but I don't understand why
one would test a UI. It seems so trivial that I would have to check that if I click a button, a specific function would run - at least
that's my impression of it - seems like duh to me.
6. **CSS Class Binding.** I've done this many times, but I keep on forgetting how to do them. It seems easier since version 4, though.
So far, I've only come across the situation where I needed to apply a class to an element if a logical condition is met. There are more
scenarios with this that I need to explore.
7. **DI beyond just constructor injection.** I understand that declaring a service at the app module level will allow me to inject it
on any component's and service's constructor. How about creating a static instance of an object (maybe like a configuration object) and
making it available for use from any component? I may have done this before, but I don't remember how. Also, I don't know why we always
marked a DI'd constructor parameter as private, plus the fact that we can specify public or protected, and they seem to all work the
same way.
8. **Directives.** I've done this in Angular 1, but since components came into play, I totally ignored directives - what they are and
why I would still need them.
9. **Nested Routes.** This is something that I blatantly ignored because I was always working with top-level administration apps whose
routing only needed to dynamically change one rectangular area. I don't even know the concept of nested routes, let alone how to set it
up and any of its reprecussions.
10. **Changing the Route Programmatically.** I know about navigating to a different route programmatically, but I don't know, for
example, how to handle when a user turns the page on a paginated result set. I always just modified the DOM, but I'm supposed to also
modify the route. How would I go about this?
11. **Modules besides the app module.** Modules are feature that helps with project organization, and may be packaged and deployed to
npm. I've done this before, didn't fully understand it, so I'll go ahead and relearn it.

Also, I would like to know how to do the following, if possible:

1. **Data-bind public service properties to elements.** From what I know, only immediate properties of a component may be data-bound to
the UI elements of that component's template (html markup).
