# My Writing vs. Markdown

I've used Markdown to save the text for my blogs in the past, and have enjoyed the fact that it's decoupled from a client that would render it. I used to store blog
text in plain html, but that meant ever only web pages can render my text. Markdown does a great job for the basics like headings, links, and images. But my writing
needs more than just headings, links, images, and paragraphs. I have quite a few semantics that I would like rendered consistently like a pitfalls block, where the
text inside would indicate warnings, common mistakes, or things that people often overlook. On an example web page, I would have liked to display a pitfalls block
in a panel with a faint red background with some dark red text, with an icon to the side. No, I would not specify graphic details in the Markdown - just the fact that
my intent is to warn. But there is no Markdown syntax for what I want, and of course, current Markdown renderers wouldn't know what HTML to convert that to when they
encounter it - it will be treated like regular text.

I see that organizations have their own "flavor" of Markdown, like GitHub, but I want my own. It looks like I will have to roll my own Markdown parser and HTML
renderer, which is a very time-consuming thing for me to develop. The other option is for my application to allow me to save a blog in "data blocks", where, for
example one data block is Markdown, the next is a pitfalls block, then the next is Markdown again, followed by a small image gallery. I'll have to save those blocks
as a JSON array of data blocks. But then, I would need to write a component that would read my data blocks array and render it to the page. The latter seems easier
because it saves me the effort of creating the parser.

## My Own Markdown Parser?

So, I have two options. The first one is, I believe the tougher one, and that is to create my own Markdown parser and renderer. This way, I can establish any syntax
I want and interpret it however I want. This is quite tough to pull, though, since I hate parsing strings (AKA, get down to the level of each character and keep track
of multiple indexes at a time. That's the tough part - the parsing. I would then collect things into a tree or array of objects corresponding to each Markdown rule,
and from there, output my HTML or whatever other markup language is needed. I believe that would be the easy part. Another, IMO, extremely challenging task is to
implement syntax highlighting myself. I would like to write actual programming language code blocks in plain text, and it would render on a web page with colored
text, a different color for keywords, comments, control structures, type names, etc. And no, I wouldn't write the color data on the Markdown text. How would we
implement such a thing?

## Data Blocks

I would consider a data block to be an object that contains just regular Markdown text, or one that contains any data structure I want that contains properties whose
value I would like to eventually display on a screen. This is a simpler approach, but this leads to a more complicated UI. There would be multiple Markdown editors,
one for each markdown block between the other data blocks. I would also need to give the user the capability to reorder and delete data blocks. I would need to write
components for every single data block - one to input, and the other to render. It's true that there isn't the nitty-gritty mind-bending string parsing, but there is
quite a lot of work that needs to be done.

## What is My Decision?

This is a tough one. The easiest decision is neither of the two. I would settle for one and only one markdown block per article, and give up a more expressive and
semantic way to render my intentions (warn, give tips, assure, etc.) I am gearing more towards making my own Markdown parser, and if I ever want to get that going,
it will be in the development and testing phases for a very very long time.