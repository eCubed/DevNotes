# Adding Atom Plugins

I'm going to start adding plugins that are for plain HTML, CSS, and JavaScript for now. I'll add Angular-related stuff later.

Out-of-the-box, I do get "Intellisense" for html elements, their attributes, and some attribute values if there is a finite set of
allowable values. I also get auto-indentation, which is nice. But I don't get html closing tag autocompletion, which is a huge overhead
for me if I have to manually type it (like I forget to do it, or I mismatch the spelling with the opening tag, both of which are
difficult to debug). Also, I don't get html linting, which I would love to have.

Typically, the ability to add the plugin is on the Welcome screen. Suppose that we closed it. Where do we find it? It's under
File > Settings (or Ctrl + ,) > Install (tab). From there, we can type what we want in the search box or check out featured packages.

The html tag closing plugin we want is autoclose-html. When it turns up in the results, we click the Install button. If we ever want
to see what packages are installed in our Atom, it's on the Packages tab, sibling of the Install tab in File > Settings. From there,
we can uninstall, disable, or configure the package.

Ah, so now, our html tags close. autoclose-html is smart enough to put the closing tag in the right place - two lines below the opening
tag (with the cursor in the line between, already tabbed) for block tags, right after the opening tag for inline tags (like span), and
will add a `/` character right before the closing angle bracket for tags that don't have closing blocks, like input. Nice!

Let's head over to CSS. We want to be able to choose colors and see what the color looks like, so we need a color picker. The most
popular one is color-picker. To activate it once it's installed, we place the cursor where we want to declare a color, and hit Ctrl + 
Alt + C. The dialog is self-explanatory, but be careful because it defaults to rgb(r, g, b) instead of hex which most probably want.
It also comes with HSL and other color models depending on how one thinks of color.