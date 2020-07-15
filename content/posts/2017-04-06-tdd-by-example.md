---
categories: 
  - apprenticeship
date: "2017-04-06T15:30:00Z"
title: TDD by Example
---

This morning I finally finished Kent Beck’s *Test Driven Development by Example*. The book itself is a pretty quick read, but James encouraged me to work through both of the examples using C#. I finished the currency example during my first week, but it took me a little while longer to get around to completing the xUnit example. I’m definitely glad that I did both as it helped me to get a better handle on TDD and familiarize myself with C# in a way that I think just reading alone would not have.

Today I was pretty happy as I was able to use one of the refactoring patterns covered in the book to refactor part of my C# version of TTT. It was incredibly helpful to have a name for the type of refactoring that I was going to do. Having a name, gave me a plan, which directly led to a much less painful refactoring than I initially anticipated.

In this case, I had a parameter that I wanted to change from an single object to a list of objects. The class that needed to change was used in lots of places, since it involved validating console input. In the past, I would have probably just changed the method signature to use the new parameter and then tried to fix the inevitable wreckage in one long painful refactoring. Instead, I tried to follow the *Add Parameter* and *Migrate Data* patterns described in TDD by Example. This was definitely a much more incremental process, than the "just blow everything up" approach. Little by little, I made very small changes until the old object was redundant. However, it wasn’t just the incremental way the changes were made that was different, each step was very strategic about the particular change to make, always keeping passing the tests in mind.

In retrospect, I guess it’s obvious, but having a systematic way to attack a particular refactoring made the work go more smoothly. Also, having the tests to fall back on gave me confidence to make the change in the first place. Initially, I was resisting making the change even though I felt like it was probably necessary. Without the tests, I could definitely imagine myself continuing to try to contort my previous design to accommodate the new requirements, but having the tests made it much easier to convince myself to make the change immediately. 

Experiences like this definitely sell me on the benefits of automated testing. Already, I’m finding myself in fewer situations in which some widely used component becomes virtually untouchable only because it’s used throughout the codebase.
