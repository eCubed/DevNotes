# Python Programming Curiosities

I'm going through the tutorials on [w3schools.com](https://www.w3schools.com/python) and I'm coming up with some questions that I don't think the tutorial covered.

1. Are primitives (numbers, strings, booleans) pass-by-value and others pass-by-reference just like in JS and C#?
   This is a tricky affair in Python because variables are not regarded as primitive or complex. Yes, in JS, Java, and C#, primitives are automatically pass-by-value, and anything else is
   pass-by-reference.

   In Python, everything **is pass-by-reference**, EVEN those that we regard as primitives like numbers, strings, and booleans. So, from inside a function with an integer parameter for example,
   that integer passed in **is the same exact one** that the caller "is talking about". This way, the runtime doesn't actually allocate another place in memory that will contain the same value,
   that is only for the function call to manipulate. NOW, if we change the value of the passed-in integer inside the function, then the copy is made, and that variable's scope then belongs to
   the function call. The function can change that value, but the caller's scope's value for that passed-in integer **remains unchanged**.

   Now, consider something like a list. When passed into a function, we're passing the list itself, because everything is pass-by-reference. Now, from inside the function, we can manipulate that
   list, such as adding and removing items on it, or modifying its contents, and as expected, the change will be reflected everywhere. HOWEVER, things get different in Python vs C#, JS, and Java
   when we instantiate a new list onto that passed-in list. In C#, JS, and Java, that new list instantiation will be reflected everywhere, even outside the function. In Python, NO. The original
   list doesn't change, and a new location in memory will be allocated for that variable, and this new memory location will only apply from within that function call's scope!

2. Does Python have interfaces as in Java, C#, and C++? 
   No. However, we can use inheritance to simulate interfaces. Also, Python allows multiple inheritance, so we can "implement" multiple interfaces. Then, we need to be careful!
3. Does Python have generic type parameters? 
   No
4. Are global variables available throughout ALL .py files in a project, or just within that one file?
   For now, I CANNOT have an answer for this. However, I have some guidelines I think would work:

   1. I will declare global variables ONLY on the main .py file.
   2. I will NEVER access global variables from ANY function of a class or on a different .py file. I could pass in global variables to those functions, though.
   3. All functions in non-main .py files will have ZERO knowledge of their surrounding scope, or any application. This way, they can be readily used anywhere, even in other programs.

5. Does python resolve the situation where we have 2+ modules in our program that we import into the main module, and they all import some of the same modules?
   Yes, it does.
6. For a dictionary type, is `myDict.key` valid and equivalent to `myDict["key"]` like it would be in JS?
   Not readily. It's an advanced topic in Python, and there's a [Stack Overflow](https://stackoverflow.com/questions/2352181/how-to-use-a-dot-to-access-members-of-dictionary) that addresses it.
   For now, let's stick to the original intentions of Python, and access properties by string index.
8. Does Python have multithreading?
   Yes