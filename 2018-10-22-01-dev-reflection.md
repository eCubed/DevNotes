# Dev Reflection for Today

Over the last two weeks, I have focused all Angular efforts on animation and directives. I have learned a great deal about them, however, there are some obstacles with
them that are taking way too much time to overcome, especially the necessary workaround for getting the `:leave` state to work when we finally remove a dynamically-
created component. I wanted to get even deeper into creating my own widgets using Angular directives to mimic some of what Bootstrap and Angular Material offer, however,
that is taking way too much time. I'll still keep the widgets effort, but I'll lower its priority.

# Back to Angular Material and a Fresh View on Responsive Design
I currently need to put out my personal website soon, and developing Bootstrap and Angular Material from scratch doesn't seem to be a wise endeavor at this point. I will
adopt the basic look of Angular Material for now, and I'll customize it later.

If Angular Material is for me to avoid having to create basic widgets, I will need something to take care of Responsive Design. I have in the past played around with
media queries and got even more confused when resolutions and dpi were thrown into the mix. I also leveraged Angular services to listen for window resize changes so that
components can listen in on whether the viewport is small, medium, or large, and then use that information to layout themselves (which includes applying different CSS
classes per viewport size, or even hiding or removing elements depending on viewport size). That service approach only considers pixel width, and is only tested on a
desktop browser. I recently came across Angular Flex Layout, which is supposed to be utilizing much of the newer CSS Flexbox framework. Flexbox, in its purest form was
very confusing to me, and couldn't get it to work right even on the most basic layout efforts. Hopefully, Angular Flex Layout will hide all that confusion and its API
is clear enough so I can just use it easily with expected results.

# Abandoning the Nitty-Gritty of Angular Development?
At this point, I need to get out of exploration mode since I need to quickly put up front-end websites. I'll just consider Angular directive-related pursuits as "nice to
know", and will only develop directives when a project calls for it. If I have time and energy leftover, I'll look even deeper into Angular directives.