# Python - Getting Started

I downloaded from [here](https://www.python.org/downloads/). I chose the most recent version, which is 3.8.1 as I'm writing this.

At the start of installation, I chose the easiest option (I don't remember what it was called), not the one where I get to nit-pick modules because I'm just starting!
I also checked the "Add Python 3.8 to PATH", which is important!

OK, so it's installed. Now what? Time to Learn.

## Visual Studio Code

I want to use VS Code because it's just an all-around awesome text editor.

So, I went ahead and created a folder somewhere on my machine that I regard as a "Python Project". I opened VS Code to this folder, and I created a python file with the extension
.py. VS Code does prompt me if I want to install any tooling for Python, so I just said yes to all of them.

## The Hello World Program

It only has one line - `print("Hello World!")`.

I like to work from inside VS Code including running the program at the command line. At first, though, running `python helloworld.py` resulted in an error that said `python` wasn't recognized.
I restarted VS Code, opened to my project's folder, and ran the command again at the command prompt, and now it works! This is quite a stupid phenomenon - can't run `python` from inside VS Code
on the first time I ever opened a project. All it takes is to close VS Code, then reopen it to the project folder, then `python` will be available from its command line.

## Basic Variables

One thing I learned is that we declare variables in Python WITHOUT specifying the type. The runtime will just infer the types and would throw errors if we didn't use them write. For example, using the
`+` operator in an attempt to concatenate a number and a string would result in an error (though this is allowed in C# because on there, the language provides overloads for the `+` operator among
primitive types).

## The Rest

I also noticed that there are no semi-colons at the end of each statement, and also, indentation defines scope, so no curly braces that I'm accustomed to. There's also much syntax I need to learn,
and I'll approach that "lightly", which I mean on an "as needed" basis. That isn't so bad, because I'll look it up the first time, and probably a few more times, until I learn them. Once I get
comfortable, I'll pursue functions, reference vs. copy, I/O, OOP, multiple files for organization, and then installing 3rd party libraries and using them.

As for now, though, I do NOT know anything useful I would want to create in Python. I've read briefly on what Python is used for, and I'm seeing its usefulness in Deep Learning, AI, and task
automation. Well, I'll keep learning and experiencing.




