# Interfaces and Classes

When C#, et. al., developers first saw TypeScript, they were so relieved that there are now classes and even interfaces in front-end development.
And yes, it is true that TypeScript classes and interfaces *can* be used and implemented exactly as in C#. However, there will be a lot of code
that we swear could violate OOP principles.

## Dynamic Keys

We are used to declaring the precise properties of that an interface should have. However, it is possible to declare an interface that can have
any number of properties, and each of those properties can have any name:

```javascript
public interface IOneWithDynamicKeys {
    [key: string]: Object[];
}
```

### Critique
It seems like we're creating a useless interface. If it can have any property, or any number of properties, and each property can be of any type,
then there really isn't a contract. It would seem like any class can implement such an interface. When we're thrown an object that is of an interface
type, we look for specific properties and functions to access from it. But when we have an interface that can have any set of properties, then
we really can't treat the interface-typed instance given to us uniformly.

It is said that we would want this interface because an incoming object from an API call could have an arbitrary set of property-value pairs. For
one, we are supposed to know exactly what the property values of that object returned to us are so we could create a matching interface or class
that reflects it. Why create a single interface that we would want to catch any object into if we can't really do anything with the instance
if we can only treat it as an `IOneWithDynamicKeys`? We might as well just keep the JSON returned to us as a plain old JSON object and not
even type attempt to type it.

## Interface as Data Types Hash Table?
We are used to declaring properties of an inerface. We can also apperently use an interface to associate strings with various data types:

```javascript
interface ISomeTypeHashTable {
   'ofTypeA' : TypeA;
   'ofTypeB' : TypeB;
   'ofTypeC' : TypeC;
}
```

### Critique

Of what use is such a "hash table"? Actually, a hash table is a collection of key-value pairs, and this interface isn't even about key-value pairs.