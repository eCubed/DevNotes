# Tooltip with Attribute Directive

We will attempt to create a tooltip directive where the user would hover over the element or component, and then a rectangular area would pop up containing 
simple text content. Then the popup would disappear when they mouse-leave. The pop up would also be positioned right above the element the directive is applied to.

Eventually, we would like to apply our `libTooltip` directive this way:

```html
<p>
  The quick brown fox jumps over the head of the lazy
  <span libTooltip="Man's best friend">dog</span>
</p>
````

For now, we will only allow text content for simplicity. We'll later modify the code to allow the popup to contain HTML content from a template.

## Tooltip Component

We need to first create a component that will contain the tooltip content.

```html
<div class="tooltip-container" [ngStyle]="{top: top, left: left}">
  <ng-content></ng-content>
</div>
```

```javascript
import {
  Component,
  OnInit} from '@angular/core';

@Component({
  selector: 'lib-tooltip',
  templateUrl: './tooltip.component.html',
  styleUrls: ['./tooltip.component.scss']
})
export class TooltipComponent implements OnInit {
  
  top: string;
  left: string;

  constructor() { }

  ngOnInit() {
  }
}
```

```
.tooltip-container {
  position: absolute;
  background-color: black;
  color: white;
  box-sizing: border-box;
  padding: 10px;
}
```

With our starting code, we need to pay attention to the following:

1. The `TooltipComponent` is a plain div element that wraps `<ng-content>`, which could be any element, set of elements, template ref, or component. We do not
know at this point, and it's alright that we don't know. For now, it will be plain text content. We will see later that `TooltipDirective` will dynamically
instantiate a `TooltipComponent` and feed into it the text that will eventually replace its `<ng-content>`.
2. The `TooltipComponent` will be physically positioned on the screen, relative to the directive's host component. We keep string references to the `top` and
`left` coordinates because we will `[ngStyle]` bind it to the container div whose class name is `.tooltip-container`. Later, we will see that upon dynamic
instantiation of `TooltipComponent`, it will be fed with information containing the host's global position coordinates. We will calculate the top and left positions
of the tooltip based on the information given to us by the directive.
3. In order for this tooltip to be positioned correctly, we need to set `.tooltip-container`'s positioning to absolute, as shown in the SCSS.

The `TooltipComponent` code is intentionally incomplete at this point. It will make more sense to complete it after we complete the directive for it.

## The Tooltip Directive

We are now ready to code the `TooltipDirective` that gives any element or component the capability to show a tooltip for it.

```javascript
import {
  Directive,
  ViewContainerRef,
  Input,
  ComponentFactoryResolver,
  Renderer2,
  ReflectiveInjector,
  HostListener,
  ElementRef } from '@angular/core';

import { TooltipComponent } from './tooltip/tooltip.component';

@Directive({
  selector: '[libTooltip]'
})
export class TooltipDirective {

  constructor(
    private elementRef: ElementRef,
    private viewContainerRef: ViewContainerRef,
    private componentFactoryResolver: ComponentFactoryResolver,
    private renderer: Renderer2){
    }

  @Input("libTooltip") tooltipContent: string;

  @HostListener('mouseover')
  onMouseOver() {
    
    /* We'll create the tooltip popup here. We will go through it later.
    */
  }

  @HostListener('mouseout')
  onMouseOut() {
    this.viewContainerRef.clear();
  }
}
```

Let us first investigate the injected parameters in the constructor.

`elementRef: ElementRef`. This gives us a hold of the directive's host element. We will need the host element's global position on the page to position the
`TooltipComponent` that will pop out.

`viewContainerRef: ViewContainerRef`. This is the agent that will officially and ultimately draw our `TooltipComponent` with the desired content passed to it.

`componentFactoryResolver: ComponentFactoryResolver`. We will need this to be able to dynamically instantiate the `TooltipComponent`. It can be thought of
as instructions for the `viewContainerRef` on which component to instantiate.

`renderer: Renderer2`. This is practically the Angular way of generating DOM elements in Angular code. We will need to generate the element that will contain
the string to show on the tooltip. This text element will actually be the element to replace `TooltipComponent`'s `<ng-content>`.

We have the `@HostListeners` set up for `mouseout` and `mouseover`. For the `mouseout`, we simply clear the `viewContainerRef`, which will remove the
`TooltipComponent` when the user's mouse leaves the host. Most of the action goes on inside the `mouseover` function.

### Making the Tooltip Pop Out

```javascript
  @HostListener('mouseover')
  onMouseOver() {
    
    let factory = this.componentFactoryResolver.resolveComponentFactory(TooltipComponent);
    const injector = ReflectiveInjector.resolveAndCreate([
      {
        provide: 'tooltipConfig',
        useValue: {
          host: this.elementRef.nativeElement
        }
      }
    ]);

    if (typeof this.tooltipContent === 'string') {

      let stringElement = this.renderer.createText(this.tooltipContent);
      this.viewContainerRef.createComponent(factory, 0, injector, [[ stringElement ]]);

    }
  }
```

There is quite a lot going on to properly pop out the `TooltipComponent`. First up, we'll need to establish the fact that we are eventually going to instantiate
a component. This is the job of the first line `let factory = this.componentFactoryResolver.resolveComponentFactory(TooltipComponent);` We pass to
the resolver's `resolveComponentFactory` the class name of the component we want to instantiate. We get a factory back that knows how to instantiate a
`TooltipComponent` for us.

We know that we will need to pass information about the directive's host to the component we're about to instantiate. The save way to do that is via an injector.
We can think of an injector as a collection of key-value pairs that we get to fill, that the component about to be instantiated can access from its constructor.
The directive will feed the injector to the instantiated component, and we haven't yet coded `TooltipComponent` to receive it. We'll address this later in the
next section. Right now, we are registering an object with the key `tooltipConfig` with the function `ReflectiveInjector.resolveAndCreate()` that takes in
an array (the collection we just talked about). Each registration is an object that has the properties `provide`, which is the key, and `useValue`, which is the
object associated with the key. In our case, the object contains only one object - the host, which is the native element of this directive's host. This way, we are
intending to pass the host element itself to the component so that it knows where to draw itself.

Notice that we have an enclosing if statement that checks for the type of `this.tooltipContent`. This is trivial right now - of course `this.tooltipContent` is
a string because we have declared so. However, we will want to do this for near-future use for when we also would like `this.tooltipContent` to also possibly be
a TemplateRef or another component. For now, we handle the case where `this.tooltipContent` is exactly a string. For this, we create a text "element" dynamically
with the `renderer`.

We now have all of the information available to us to instantiate the `TooltipComponent`. `viewContainerRef.createComponent` really has five parameters, and
we'll need to use the first four. The first parameter is the factory, which instructs the `viewContainerRef` to construct a `TooltipComponent`. The next
parameter is the index position of the component, which, for our purposes is 0 because it will be the only component we wish to add. The third parameter is the
injector which will allow the component to access any data we register into it (the injector). The fourth parameter is exactly what will replace the component's
`<ng-content>` tag with whatever element or elements we want. This fourth parameter is a double array, and we'll just accept it that way, not getting into the
reasons why. We only have one element we wish to replace `<ng-content>`, so that is exactly the `stringElement` we created just a while ago. So, we wrap the
double brackets around it to denote that we are passing in a two-dimensional array with only one element.

We are now ready to rework the `TooltipComponent` to receive the host's native element.

## Reworking TooltipComponent

Right now, the `TooltipComponent` doesn't know where to draw itself, and we will need to make modifications to have it appear exactly where we want.

```javascript
import {
  Component,
  OnInit,
  Inject,
  ElementRef } from '@angular/core';

@Component({
  selector: 'lib-tooltip',
  templateUrl: './tooltip.component.html',
  styleUrls: ['./tooltip.component.scss']
})
export class TooltipComponent implements OnInit {
  /* We will obtain positioning of this component from the directive!
  */
  top: string;
  left: string;

  constructor(
    @Inject('tooltipConfig') private tooltipConfig,
    private elementRef: ElementRef) { }

  ngOnInit() {
    const { top, left } = this.tooltipConfig.host.getBoundingClientRect();
    const { height } = this.elementRef.nativeElement.getBoundingClientRect();

    this.top = `${top - height - 20}px`;
    this.left = `${left}px`;
  }

}
```

Back in the directive, we passed an injector along with the component's factory in order to instantiate the component. If we recall, we registered an object with the
key `tooltipConfig`. Now, we're going to receive that object in our component. We have in the constructor `@Inject('tooltipConfig') private tooltipConfig`,
which is our component really requesting the object whose key is `tooltipConfig` from the system. We now have access to that host, and so, in `ngOnInit()` we
get the host's top and left coordinates as starting points to locate our component: 

```javascript
const { top, left } = this.tooltipConfig.host.getBoundingClientRect();
```

But we also need the height of our component so we can raise it higher on the screen so that its bottom will be right on top of the host element. In order to do this,
we inject an `ElementRef` in the component's constructor. Now, we can get its height:

```javascript
const { height } = this.elementRef.nativeElement.getBoundingClientRect();
```

Now, we can calculate the component's final coordinates:

```javascript
this.top = `${top - height - 20}px`;
this.left = `${left}px`;
```

For the component's top, it's the host's top minus the height of the component, and another minus 20. This minus 20 accounts for the 10px padding for the top and
bottom of the component.

## Next Steps

We can further enhance this component in many ways. For example, what if we wanted the user to specify different locations for the tooltip, like to the right, left,
or below the host component? For that, we will have to register another property to `tooltipConfig` called position. We would then read that position from
inside the component, and depending on what it is, calculate different values for the component's top and left coordinates.

We have also only handled the case where the tooltip is a single string value. We will later need to allow our directive to recognize TemplateRef and other
components as well.

This component has a popup, and many other components are popups by nature, such as context menu, date picker, any picker, etc. We now have the experience with
directives to make popups happen including controlling where they would pop up. However, many of those popups will require some sort of user input, and will
somehow need to let the directive know what the user input and when the user finished providing that data.