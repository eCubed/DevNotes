# Angular Directives Exploration

In my Angular 2+ development efforts so far, I have avoided custom directives. I used built-in directives like `*ngIf` and `*ngFor` only for my apps to work,
never really diving deep into how they work.

Today, I am looking into creating custom directives, especially ones that will add elements to the hosting element's DOM so I can do things like tooltips,
date selector, etc. I don't think that creating components that explicitly encapsulate one component and some custom DOM is viable - I'll end up creating nearly-
identical components whose only difference is the component or native html element I'm wrapping.

So far, I have learned a few things about structural directives, and I won't get into too much detail in this post about them.

* Directives get a decorator for a class, `@Directive` just like components have `@Component`. So instead of creating components, I create directives that can
be applied to any component or native element.
* I should inject `TemplateRef<any>` in my structural directives. The `TemplateRef<any>` is practically a collective handle on whatever is inside the tags of
the component or native element I'm applying my structural directive to.
* I should inject `ViewContainerRef` which is the governing object that decides how to ultimately build (or not build) what the user sees. I actually hand over
the `TemplateRef<any>` to `ViewContainerRef` if I need to show the hosted component or native element, based on some logic.
* I should inject `Renderer2` if I needed to build html in Angular code in a similar fashion to plain JavaScript. Ultimately, 