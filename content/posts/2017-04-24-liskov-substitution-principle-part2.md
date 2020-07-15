---
categories: 
  - apprenticeship
date: "2017-04-24T10:30:00Z"
mathjax: true
title: Liskov Substitution Principle Part 2
---

In [part 1]({{< ref "2017-04-23-liskov-substitution-principle-part1.md" >}}), we covered the conditions that overriding methods in a subclass must adhere to in order to be in compliance with the Liskov Substitution Principle. Depending on the language the compiler may already force you to comply with those constraints. For instance, C# requires that both the argument type and the return type be invariant for overriding methods in the subclass, while Java requires invariant arguments, but allows covariant return types [[1](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)#Inheritance_in_object-oriented_languages)]. Regardless, even if these constraints are already being enforced by the compiler I still think it’s helpful to understand why they are in place.

In this post we’ll cover the remaining conditions spelled out in the Wikipedia definition of LSP.

> In addition to the signature requirements, the subtype must meet a number of behavioral conditions. These are detailed in a terminology resembling that of design by contract methodology, leading to some restrictions on how contracts can interact with inheritance:

> * Preconditions cannot be strengthened in a subtype.
* Postconditions cannot be weakened in a subtype.
* Invariants of the supertype must be preserved in a subtype.
* History constraint (the "history rule"). Objects are regarded as being modifiable only through their methods (encapsulation). Because subtypes may introduce methods that are not present in the supertype, the introduction of these methods may allow state changes in the subtype that are not permissible in the supertype. The history constraint prohibits this. ... 

#### Assumed class hierarchy: Object \\( \geq \\) Shape \\( \geq \\) Rectangle \\( \geq \\) Square

### Preconditions cannot be strengthened in a subtype.
A precondition is a condition that must be true just prior to some section of code or method executing. In the example below, the precondition is the check that `value` is less than 1000 before setting `Rectangle's` width. For `NarrowRectangle` we've strengthened this precondition by requiring that `value` be less than 20. Client code that is able to successfully use `Rectangle`, (i.e. setting `value` to 500) would of course not be able to use `NarrowRectangle`, which implies a violation of LSP. While the precondition cannot be strengthened, it would be permissible to weaken it in the subclass.
``` java
class Rectangle
    void SetWidth(int value)
       if (value > 1000) { throw ArgumentException }
       width = value

class NarrowRectangle extends Rectangle
    void SetWidth(int value)
       if (value > 20) { throw ArgumentException }
       width = value
```

### Postconditions cannot be weakened in a subtype.
The example below is from _PPP in C#_. In the subclass we’ve adapted `Square` so that `setWidth` and `setHeight` methods additionally set `height` and `width` , so that the two properties remain equal. However, the base class implies a postcondition for both `setWidth` and `setHeight` (`width == value && height == previousHeight`, and `height == value && width == previousWidth`). Therefore, it’s reasonable for users of `Rectangle` to assume that these two attributes can be modified independently and thereby make assertions like below. However, this assertion would fail for the `Square` class. Failing to be able to substitute a `Square` for a `Rectangle` again signals a violation of LSP.

``` java
void testArea(Rectangle rectangle)
   rectangle.setWidth(4)
   rectangle.setHeight(5)
   assert(rectangle.area() == 20)
```


``` java
public class Rectangle
    void setWidth(int value)
    	width = value

    void setHeight(int value)
    	height = value

class Square extends Rectangle
    void setWidth(int value)
       width = value
       height = value

    void setHeight(int value)
       width = value
       height = value
```

### Invariants of the supertype must be preserved in a subtype.
An invariant is a condition that can be assumed to be true during the execution of some portion of the program. In order to comply with LSP a subtype can't weaken any of the parent class's invariants. In the example below users of `Rectangle` can assume that `rectangleName` never violates `rectangleName's` length invariant during the execution of `setName`. However, `NarrowRectangle` weakens this invariant by allowing the invariant to be violated for a period of time. Clients of `Rectangle` can no longer read `rectangleName`during the execution of `setName` with the assumption that `rectangleName's` length constraint is not violated.

``` java
class NameTooLongException extends Exception
class Rectangle
   void checkInvariant()
       if (Length(rectangleName) > 20) {throw NameTooLongException}

   void setName(string name)
       oldName = rectangleName
       checkInvariant()
       rectangleName = name + "Rectangle"
       checkInvariant()
       // do more stuff
       rectangleName = oldName
		
public class NarrowRectangle extends Rectangle
    void setName(string name)
       oldName = rectangleName
       checkInvariant()
       rectangleName = name + "Rectangle and a bunch more stuff"
       // do more stuff
       rectangleName = oldName
```

### History constraint (the "history rule").
Since subtypes can introduce methods that are not present in the parent class, it's possible for them to introduce methods that cause state changes that are not permissible in the parent class. In the example below `name` is not intended to be modified as there is no publicly available setter for `Rectangle`, however `OutlawRectangle` introduces a setter, which allows for `name` to be modified in a way that the parent class cannot be. Again this is a violation of LSP as clients of `Rectangle` can no longer assume that `name` is immutable.

``` java
public class Rectangle
    string GetName()
        return name;

public class OutlawRectangle extends Rectangle
    void ChangeName(string newName)
        name = newName;
```

### LSP in practice
Substitutability of subclasses for base classes is important if we want to be able to write general, broadly applicable code. Without substitutability it becomes very difficult to adhere to the other SOLID principles, since things like type checking become necessary in our methods. For many statically typed languages the compiler will protect you against some of the LSP violations covered in these posts (contra / covariance of method arguments). However, this will vary from language to language so it's good to be aware of how you can possibly violate LSP regardless of how helpful your current compiler is. 

Finally, LSP is really about thinking very carefully about how clients will use our objects and methods and the assumptions they will make about them. Given that we can’t fully know this in advance, often it’s not possible to protect against all LSP violations without introducing massive, potentially unneeded complexity. However, by carefully considering how our classes will be used or are being used, we should be able to anticipate design choices that would grossly violate LSP.