# TypeScript Types

We know that TypeScript has primitive types of boolean, number, and string, and we can declare properties to be of those types as usual in JavaScript
and any other programming language. We can also create classes with properties of primitive and complex types, and instantiate objects of our
classes. There are a lot that TypeScript allows that are not allowed in programming languages such as C#, C, C++, Pascal, and Java, AKA, "classic"
and often backend programming languages. Throughout this series, we will catalog a lot of the "anomalies" that we see and examine them.

## Type ORing

We are used to declaring an object, class or inheritance property, parameter type, or return type to be exactly of one and only one type, which could
be of a class or an interface. Typescript allows us to declare something *to be able to be of TypeA or TypeB*. For example:

```javascript
export class Something {
   data: TypeA | TypeB | TypeC;
}
```

In the above example, `TypeA`, `TypeB`, and `TypeC` are distinct classes that may or may not share a common interface. Such a construct is
allowed. Also, we can do this:

```javascript
function someFunction(data: TypeA | TypeB | TypeC){
    ...
}
```

And as a return type of a function:

```javascript
function someFunction(paramA: string, paramB: bool) : TypeA | TypeB | TypeC {
    if (paramA === "a")
        return new TypeA();
    else if (paramA === "b")
        return new TypeB();
    else if (paramA === "c")
        return new TypeC();

    return null;
}
```

Furthermore, we can even mix primitives with complex types in "Type ORing":

```javascript
export class AnotherSomething {
   data: TypeA | TypeB | bool | string;
}
```

There is also another common variant that something is either one of a single type or an array of that type of object:

```javascript
export class AnotherSomething {
   data: TypeA | TypeA[];
}
```

Perhaps this is due to the fact that TypeScript will ultimately be transpiled to JavaScript, which is a loosely-typed language. However, we always
had the impression that TypeScript would help us program with strict typing in mind to make it easier to develop, since we *do* get code completion
and we get errors when we try to access a property that doesn't exist in a class with TypeScript.

### Critique

Why would we want a property to be of either one of a few declared types?

We typically would have classes `TypeA`, `TypeB`, and `TypeC` declared and they would share a common interface `ICommonInterface`.
This way, we can easily declare property, parameters, and return types to be that of their common interface `ICommonInterface`. This way, 
the object would be automatically either one of the declared types that share the common interface, and in code, we should treat that object
*as an* `ICommonInterface` instead of one of their possible concrete types.

How would we handle in code, an object that could be one of the few declared types? It seems like we would have to perform reflection and/or
casting which would complicate things.

## Anonymous On-The-Fly Type Declarations

When we declare a property of a class or interface, its type can literally be the same as the body of a class or interface:

```javascript
interface ISomeInterface {
    person: {
        firstName: string,
        lastName: string,
        age: number,
    };
}
```

We could also declare an array of an anonymous on-the-fly type:

```javascript
interface ISomeInterface {
    people: {
        firstName: string,
        lastName: string,
        age: number,
    }[];
}
```

or

```javascript
interface ISomeInterface {
    people: [{
        firstName: string,
        lastName: string,
        age: number,
    }];
}
```

We are not sure what the difference is between the two array notations, but both can be done.

### Critique

Why can't we actually just declare a `Person` class to avoid this extra typing? Somewhere down the line, we will need to write a function
that takes a `person` as a parameter, then we'd have to declare the function parameter this way:

```javascript
function personProcessor(person: { firstName: string, lastName: string, age: number}){
    ...
}
```

This results in a lot of characters on the screen. Perhaps what this means is that a person in `ISomeInterface` can be any object, even an 
anonymously-typed object that must have at least the three declared properties. But in OOP, we would declare an `IPerson` interface, and we must
create concrete classes that implement `IPerson` interface. This way, we could avoid a lot of extra coding.

## Type Declaration of Allowable Primitive Values

This construct does not exist in a classic programming language. We can declare a type to be a finite set of possible values of a primitive:

```javascript
type AnimalNames = 'dog' | 'cat' | 'horse' | 'cow' | 'pig';
```

If `AnimalNames` would be a parameter of a function, we would write:

```javascript
function speak(animalName: AnimalNames){
    if (animalNames === 'dog')
        Console.log("Bark");
    ...
}
```

And we would call this function this way:

```javascript
speak('dog');
speak('cat');
speak('cow');
```

The value that we pass into the function whose parameter is of that declared type MUST match the primitive type of the valid values, AND must
match one of the declared allowable values.

### Critique
While the above allowance looks handy, declaring a type which really is a primitive type for the purpose of stating a finite set of allowable values
is repurposing the meaning of type. When we think of `type`, we think of what property names and functions an instance of that type is, not
allowable values. When we want to stipulate allowable values, we typically use a hash table or use a static class with static readonly properties,
one property for each allowable value.

## Type Declaration and Type ORing

We can also use the `type` declaration to declare what would be an overarching type to be either one of a finite set of types:

```javascript
type LivingThing = Animal | Plant | Bacteria | Fungus
```

... where `Animal`, `Plant`, `Bacteria`, and `Fungus` are actual classes that may not even derive from the same superclass or implement
a common interface.

### Critique
This looks like a step above having to declare all the allowed types inline (in property type declaration, function parameter, etc.). However,
if we were to do this, all of those classes would have at least one property in common so that we could create an interface or superclass that
they would all derive from. It seems like that would be an exact equivalent of what the above `type` declaration would achieve. However, what
sense would there be, then, to put a bunch of possibly unrelated classes under one umbrella of an overarching type?

