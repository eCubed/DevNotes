# Angular Schematics Primer

In short, Angular Schematics is practically a development suite where we get to define code generation.
Out of the box, the Angular CLI comes with schematics we already use, like most commonly, `ng g component`,
where when run, will generate a folder with four files. Suppose we want something that will generate
a folder with a bunch of files, whose code is specific to our needs. We can do that with schematics.

Schematics are a very powerful development tool, and so, there are a lot of intricate and complicated
parts to it. 

The easier part of it is defining our files and folder structure. We will refer to that
file-folder structure as our template. Template code contains our usual Typescript, HTML, and [S]CSS
code, but with `<%= %>` placeholders all over the place to let us insert, for example, what we want
to name what we will generate. We will later discover the details of templating, since it involves
some string manipulation of values both inside each file and even the actual file names of each file
we would want to generate.

The more complicated part of templating is what's referred to as a `Tree`, which we will somewhat "ignore".
It feels like creating template files with placeholders for values provided at the prompt is all we need
to write, because that's all we need in Visual Studio. This is not the case with Angular Schematics.
We will "ignore" the `Tree` concept because at the time of this writing, its definition, concept, and practice
are, in our opinion, very vague and complicated, and actually still in development. There is actually
some Typescript code that we will write, whose code code reads the parameter values from the command
prompt (supplied by the user), decides what to do depending on those parameter values, go through our 
template files and finally write our files to our project. We will get into some level of detail when
we start coding our schematics, enough to do what we need to do, but not to get into the details of
the `Tree` concept.