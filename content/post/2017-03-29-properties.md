---
categories: 
  - apprenticeship
date: "2017-03-29T09:30:00Z"
title: Properties
---

It’s interesting seeing what language features repeat themselves across languages. I came across properties for the first time today in C#. It’s a concept that I’m familiar with from Python, but I thought that it was kind interesting, because the two languages have pretty close to opposite philosophies regarding access control for class members. However, by using a property Python interfaces start to behave more like C# interfaces (more restrictive) and C# interfaces start to look like the simple default interfaces in Python.

In Python it’s not really possible to make a member private. By convention members prefaced with and a single underscore signal methods and variables that relate to implementation details and therefore should not be relied upon (i.e. you’ve been warned this might change in the future). However, there is nothing to stop a user from accessing these “private” member variables. However, in C# the default for class members is to make them private, so unless explicitly stated nothing inside a class is accessible from outside of it. 

One of the nice things about properties is that you can start with the simplest possible interface for accessing member variables. However, later if more complexity is required (maybe value checking) you can adapt the internals of the class without changing the interface. Below is an example in Python.

``` python
# Normal way of accessing member variables
class Foo:
   def __init__(self):
       self.x = 5

foo = Foo()
print(foo.x)
foo.x = 6
print(foo.x)

# Now using a property. Notice the interface doesn’t change.
class FooNew:
   def __int__(self):
       self._x = 5

   @property
   def x(self):
       return self._x

   @x.setter
   def x(self, v):
       if 0 > v or v > 10:
           raise ValueError("Not a value between 0 and 10")
       self._x = v

foo_new = FooNew()
# still access the same way as first example
print(foo.x)
# still set the same way as first example
foo_new.x = 6
print(foo_new)
# but below now raises an error
foo_new.x = 20
```

And FooNew in C# 

``` csharp
using System;

namespace Example
{
   public class PropertyExample
   {
       class FooNew
       {
           private int _x;

           public FooNew()
           {
               _x = 5;
           }

           public int x
           {
               get { return _x; }
               set
               {
                   if (value > 10 || value < 0)
                   {
                       throw new ArgumentException("Not a value between 0 and 10");
                   }
                   _x = value;
               }
           }
       }

       static void Main(string[] args)
       {
           var fooNew = new FooNew();
           Console.WriteLine(fooNew.x);
           fooNew.x = 6;
           Console.WriteLine(fooNew.x);
           fooNew.x = 20;
       }
   }
}

```

I found it interesting that each language seems to implement the same feature for different reasons. In Python using a property allows for more restrictive / finely controlled access to member variables similar to C#. Whereas in C# using a property allows a for a simpler more consistent interface for accessing member variables similar to Python.

### Resources:

[http://csharpindepth.com/Articles/Chapter8/PropertiesMatter.aspx](http://csharpindepth.com/Articles/Chapter8/PropertiesMatter.aspx)

[http://stackoverflow.com/questions/2521459/what-are-the-default-access-modifiers-in-c](http://stackoverflow.com/questions/2521459/what-are-the-default-access-modifiers-in-c)

[https://docs.quantifiedcode.com/python-anti-patterns/correctness/implementing_java-style_getters_and_setters.html](https://docs.quantifiedcode.com/python-anti-patterns/correctness/implementing_java-style_getters_and_setters.html)

[http://stackoverflow.com/questions/2627002/whats-the-pythonic-way-to-use-getters-and-setters](http://stackoverflow.com/questions/2627002/whats-the-pythonic-way-to-use-getters-and-setters)

[http://blaag.haard.se/What-s-the-point-of-properties-in-Python/](http://blaag.haard.se/What-s-the-point-of-properties-in-Python/)

