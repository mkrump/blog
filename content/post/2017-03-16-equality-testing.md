---
categories: 
  - apprenticeship
date: "2017-03-16T17:30:00Z"
title: A.Equals(B) ... Wait why not!?
---

Today I finished working through the currency example in _Test-Driven Development by Example_. The book is in Java, but I’m following along in C#, so most of the code is very similar, but sometimes something breaks and I off I run trying to figure out what went wrong. In the final chapter as I was getting ready to wrap up, one of my earlier tests failed and I was left scratching my head as to what was going on. Having only worked with dynamic languages in the past (Python specifically), I’m used to thinking about objects more in terms of their methods and attributes rather than their explicit type. For example, If you look at the section on [creating a new estimator](http://scikit-learn.org/stable/developers/contributing.html#rolling-your-own-estimator) in scikit-learn (a very popular machine learning library in Python) you can see that all you have to do to build your own custom estimator is implement the appropriate interface. If the appropriate methods and attributes are there you’re good to go. With C# I’m learning that you have to be a little more explicit about what you’re doing, but the benefit is that your tools are much more helpful, and foreign code seems much easier to read (first impressions here obviously).

The snippet of code and the accompanying test that were giving me problems are below. The failed test output made the objects look like they were identical. I checked all of the attributes and everything was the same. Calling `.GetType()` evaluated to `Money` for each object. I suspected that even though the object returned from `Times` was clearly a `Money` object the comparison was never being made, since according to the function signature `Times` returned an `Expression` interface not a `Money` object.  Finally, after some looking around I realized that I needed to override the `Equals` method in order to compare objects of different types (Equals was already defined for objects of the `Money` class). Once I did that and included an explicit type cast the test was resolved. 

Josh on his way out the door (**Thanks Josh!**) gave me a very helpful overview of how the type chain resolves, and I think that I generally understand, but understanding how the type system actually works is definitely something that I need to spend some more time with.

### Money Class
``` csharp
public class Money : IEquatable<Money>, Expression
{
...
   public Expression Times(int multiplier)
   {
   return new Money(Amount * multiplier, Currency);
   }
...
}
``` 
### Test
``` csharp
public void TestMultiplication()
{
   Assert.Equal(Money.Dollar(10), five.Times(2));
}
```
### Updated Equals methods
``` csharp
public class Money : IEquatable<Money>, Expression
{
...
   public bool Equals(Money other)
   {
      Money money = other;
      return Amount == money.Amount &&
            Currency == money.Currency;
   }

   public override bool Equals(object other)
   {
      Money money = (Money) other;
      return money != null && Equals(money);
   }
...
}
```
