---
layout: post
title:  Interfaces
date:   2017-03-24 09:30:00 -0500
categories: 
  - apprenticeship
---


One of the new stories for my C# implementation of TTT was to allow users to choose from a menu of console based UIs (e.g. retro UI, color UI, sparkles UI, etc.). This forced me to think a bit about the general sorts of things that all TTT UIs should do. I came up with a short list of methods along the lines of `RenderBoard`, `RenderMsg`, and so on. But then the question is how do you actually describe and enforce this design for the existing UIs, while at the same time communicating these requirements to developers interested in making their own UI. After a bit of research it appeared that an Interface was exactly what I looking for. Interfaces are something that I’ve not used previously. However, initially I was just mentally substituting Interface with Python’s Abstract Base Classes, which turned out to not be exactly right (I’ll come back to that in minute).

I’ve mentioned scikit-learn before, but I think it’s good for illustrating the differences in approach between Python and C#. If you’d like to create you [own estimator](http://scikit-learn.org/stable/developers/contributing.html#rolling-your-own-estimator) in scikit-learn you effectively just implement an Interface. However, the difference is the Interface is nothing more than a suggestion in scikit-learn. They’ve built tooling to check that your implementation is a valid, but the onus in on the developer to run these tests. The result is that if you miss one of the required methods or return the wrong type at some point you might not find out until much, much later on via a runtime error when the missing piece of the Interface is used. Whereas, in C# the interface is stronger than just a suggestion, it acts as a guarantee about what methods will be present, along with their associated return types.

Like I said earlier, I immediately drew analogy between Interfaces in C# and Python’s ABCs. However, C# also has ABCs. On the surface the two appear very similar, so why might you prefer one over the other? One of the main reasons seems to be a practical one. In C# multiple inheritance is not allowed, but a class can implement multiple interfaces. However, ABCs at their core are classes. They even though they can’t be directly instantiated they can contain data and provide implementations of methods, whereas Interfaces are purely virtual. The distinction as to when you’d use one over the other is still not totally clear to me, as I think I’d tend to always choose the more flexible Interface over an ABC. However, going forward hopefully I’ll get some experience using both to help better understand when each is appropriate. 

## Resources

[MSDN Interfaces C# Programming Guide](https://msdn.microsoft.com/en-us/library/ms173156.aspx)

[MSDN Recommendations for Abstract Base Classes vs. Interfaces](https://msdn.microsoft.com/en-us/library/scsyfw1d(v=vs.71).aspx)

[SO: Question: Interface vs Abstract Base Class](http://stackoverflow.com/questions/761194/interface-vs-abstract-class-general-oo)
