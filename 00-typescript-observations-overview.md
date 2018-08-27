# TypeScript Observations

The goal of this TypeScript observation series is to collect what many back-end programmers regard as unusual syntax and/or allowances in
TypeScript programming. Many C#, C++, and Java developers have embraced TypeScript for allowing the same kind of strict typing for front-end
development, and also allowing them to implement OOP concepts like inheritance, interfaces, and polymorphism. However, there are a ton of
TypeScript code that simply is not allowed in the "classic", often backend programming language. Some "syntax" is due to ES2016 features such
as destructuring and the spread operator. There are also a lot of "violating" constructs such as declaring a property to be of type A or type A[],
and declaring a property's, parameter's, or the return type (of a function) to look like the body of an interface declaration.

The goal of this series is to collect the "anomalies", find out what they mean, why they might be used, and to evaluate whether or not they are
actually necessary.