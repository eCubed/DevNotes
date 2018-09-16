# More Atom Plugins

So far, I've got autoclose-html (which I'm surprised didn't come with Atom) and color-picker. Both are awesome so far. I wonder what
other plugins are available.

One need that I can think of right now is linting. I didn't care for this before because I thought that I should be manually and
explicitly consistent with my coding conventions and syntax. I know that I get some "linting" in C# when I use an undeclared variable,
syntax errors, and other programming errors. It turns out that I don't get that automatically in Atom. I would like to see red squiggly
lines underneath things like unclosed html tags, syntax errors, and other markings that Indicate I have improper coding style (like
how I use spaces and extra lines).

## linter-jshint

So I chose the most popular JavaScript linter - jshint. I installed it, and I expected it to be a full experience much like I have with
C# in Visual Studio. It does NOT detect everything, unfortunately.

It detects syntax errors like unmatching curly braces, improper placement of semi-colons, etc, which are awesome helps. However, it was
lacking the following that I expected:

* Using a variable that hasn't been declared isn't seen. Maybe this isn't a JS issue and due to my lack of JS knowledge.
* It doesn't know objects like `document` or `array`, thus it won't yell at me if I try to access a property or function, that is
undefined for a standard JS object.
* It didn't detect inconsistencies with spacing issues (like between if/for and the opening parenthesis)

Well, anyway, I won't be working in plain JS. I will be working in Angular, but I'll set up linters for that later.

## linter-htmlhint

Like linter-jshint, there were a few things that this detected for me, like unmatching opening and closing tags, and requiring a
title tag in the head tag.

It didn't detect the following, though:

* Bogus html tags. I guess HTML nowadays can have any tag we want.
* Invalid attributes and values. I guess if it looks like html, it's html even if we have innapropriate attributes and attribute values.

It's still good help.

## linter-stylelint

This linter is for CSS and SCSS (which I'll be using more often). It will tell me things like invalid property if I put in a bogus
property like margin-diagonal, or if I put an inappropriate unit of measure for sizes and locations.

It didn't detect the following, though:

* Bogus values for legit CSS properties, like `list-style: elephant`;

## Afterthought

I guess I don't fully understand what linting exactly is and what's included or not included. After going through these first 3 linting
packages, I thought I understood it. So far, it seems like linting will pick up syntax errors (missing or extra characters). I guess
linting is not responsible for ensuring that we're accessing appropriate properties and methods of an object (like in JavaScript and
bogus tags and attributes in HTML), but in CSS, it's strict about enforcing correct CSS property names, units, and even white space.
For the white space strictness, I was hoping the same for JavaScript as I've seen in Sublime Text, but I guess not.

I'll just take linting as a tool for helping me code consistently and correctly, and not as full-featured (IMO) as Intellisense is in
VS.