# Side Projects to Work On Shortly

A couple of days ago, I thought about the possibility of creating my own markup interpreter and renderer because I wanted to add semantic data beyond those that
get rendered as native html tags. I was hesitant at first because I thought I would have to do nit-picking nitty-gritty string parsing where I would have to keep
track of so many indexes. It turns out that I can use regular expressions and string replacements to pull this off. It would be one regular expression and one
text replacement template pair per each markdown rule. I would collect all these pairs, and run the input string through all of them until all matching and
string replacements are done.

I also came across the need to syntax highlight onto HTML because I wanted to put code snippets on a blog or article. It turns out that I can use the same mechanism
to detect keywords, punctuation, data types, etc, and surround them with span tags with specific class names.

Before I get to develop this, I will need to create an Angular 6 project that would house the module that would contain all of my code to pull this off, and I would
like to be able to publish it to npm so I can use it across my apps. The problem is, I forgot how to package and publish Angular modules. I used to do this in
Angular 4 & 5, but I believe that the process is different in Angular 6+.

So I have to do the following in order:

1. Create an Angular app that would house the module for all of this. I'll call it, just for kicks, `markside`.
2. Research how to create modules in an Angular 6 app that could be individually published to npm.
3. Research how to be able to reference modules and their contents from the main application.
4. Research how to publish these modules to npm.
5. Live-test the npm installed module on a different app.