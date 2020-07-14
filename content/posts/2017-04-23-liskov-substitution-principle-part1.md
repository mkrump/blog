---
mathjax: true
title:  Liskov Substitution Principle Part 1
date:   2017-04-23 10:30:00 -0500
categories: 
  - apprenticeship
---

I'm currently reading *Agile Principles, Patterns, and Practices in C#* and recently finished the section covering the SOLID principles. The SOLID principles are something that I only had a passing familiarity with before applying to the apprenticeship program. However, during the application process and over the course of my apprenticeship, I’ve read and reread various materials related to the SOLID principles. All of the SOLID principles definitely have multiple levels of understanding, and I think that they are something you only truly internalize after using them over a fairly long period of time. With that being said, I feel like the principle that I've had the most difficulty with initially understanding is the Liskov Substitution Principle. It seems easy enough to recognize a violation after it occurs, but I feel like I don't have a great understanding as to how to protect against violations in the first place. 

PPP defines the Liskov Substitution Principle as follows:

> “Subtypes must be substitutable for their base types.”

That seems straightforward enough, but I became somewhat confused after looking at the Wikipedia entry for LSP. Wikipedia initially defines it based on the original Barbara Liskov and Jeannette Wing paper as:

> Subtype Requirement: Let \\( \phi(x) \\) be a property provable about objects x of type T. Then  \\( \phi(y) \\) should be true for objects y of type S where S is a subtype of T.

Then expands further: 

 >Liskov's principle imposes some standard requirements on signatures that have been adopted in newer object-oriented programming languages (usually at the level of classes rather than types; see nominal vs. structural subtyping for the distinction):

> * Contravariance of method arguments in the subtype.
* Covariance of return types in the subtype.
* No new exceptions should be thrown by methods of the subtype, except where those exceptions are themselves subtypes of exceptions thrown by the methods of the supertype.

> In addition to the signature requirements, the subtype must meet a number of behavioral conditions. These are detailed in a terminology resembling that of design by contract methodology, leading to some restrictions on how contracts can interact with inheritance:

> * Preconditions cannot be strengthened in a subtype.
* Postconditions cannot be weakened in a subtype.
* Invariants of the supertype must be preserved in a subtype.
* History constraint (the "history rule"). Objects are regarded as being modifiable only through their methods (encapsulation). Because subtypes may introduce methods that are not present in the supertype, the introduction of these methods may allow state changes in the subtype that are not permissible in the supertype. The history constraint prohibits this. ... 

The definition provided in PPP along with Liskov's definition summarized by Wikipedia explain how subtypes that do not violate the LSP should be structured, while the expanded definition provides a more complete roadmap for how not to violate the LSP. I thought that it would be good to go through each point to better understand what LSP is getting at. 

### Contravariance of method arguments in the subtype.

The arguments of a subtype’s methods are said to be covariant if their subtyping relation with one another is in the same direction as the parent and child class. While a method's arguments are said to be contravariant if their subtyping relation with one another is in the opposite direction of the parent and child class.

In the example below (in pseudocode), `SquareDrawer's` `draw` method would be an example of a subtype with a method for which the arguments are _**not contravariant in the subtype**_, while `ColorRectangleDrawer's` `draw` method is an example of a subtype with method arguments that are _**contravariant in the subtype**_. You can see why `SquareDrawer` would violate LSP, because an instance of it can’t be used in place of an instance of `RectangleDrawer` since its draw method only accepts a type of `Square`. However, an instance of `ColorRectangleDrawer` can be used in place of `RectangleDrawer`, since its method accepts the more general `Shape`, and `Rectangle` is indeed a `Shape`, so client code calling this method using `Rectangles` would continue to work.

#### assumed class hierarchy: Object \\( \geq \\) Shape \\( \geq \\) Rectangle \\( \geq \\) Square


``` java
class RectangleDrawer
	void draw(Rectangle rectangle)

class ColorRectangleDrawer extends RectangleDrawer
	void draw(Shape shape)

class SquareDrawer extends RectangleDrawer
	void draw(Square square)
```
### Covariance of return types in the subtype.

The second principle allows for a child class to override a parent class’s method return type to be more specific. In the example below, you can see that `RectangleGetter`  returns a more specific type than its parent class `ShapeGetter`, which is OK, because we can expect that any functionality that exists for `Shape` will also exist for `Rectangle`. However, `SquareGetter` violates the LSP because it returns the more general `Object` type, which may not behave the same as `Shape`, since there are many types of `Objects` that are not `Shapes`.

``` java
class ShapeGetter 
	Shape getShape(int id)

class RectangleGetter extends ShapeGetter
	Rectangle getShape(int id)

class SquareGetter extends ShapeGetter
	Object getShape(int id)
```

### No new exceptions should be thrown by methods of the subtype, except where those exceptions are themselves subtypes of exceptions thrown by the methods of the supertype.

More so than the first two rules, I find the third rule intuitively easier to understand. It makes sense that we wouldn't want to introduce new exceptions in a subclass, since client code designed for the parent class would probably be unaware that such exceptions exist. In the example below, `SmallShapeDrawer` adheres to this rule, since client code catching `SizeException` will also successfully catch the derived exception `TooSmallException`. However, `ColorShapeDrawer` violates the rule, as client code catching `SizeException` will fail to catch `BadColorException`.

``` java
class SizeException
class TooSmallException extends SizeException
class BadColorException

class ShapeDrawer
    void draw(Shape shape)
       if IsInValidSize(shape){throw SizeException}
		
class SmallShapeDrawer extends ShapeDrawer
    void draw(Rectangle rectangle)
       if IsTooSmall(shape){throw TooSmallException}

class ColorShapeDrawer extends ShapeDrawer
    void draw(Shape shape)
       if IsWrongColor(shape){throw BadColorException}
```

I plan on working through the remaining guidelines in a followup post.
