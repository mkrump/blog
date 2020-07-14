---
categories: 
  - apprenticeship
date: "2017-04-03T09:30:00Z"
title: static fields
---

Last week I continued to work on my TTT game in C#. The approach that my mentors and I are taking this time is somewhat different than the approach that I took for the apprenticeship submission. Before creating any `Players`, `Boards`, or other TTT specific classes, we’re creating all of the various menu and UI components. For me this feels a little uncomfortable since I consider the menu and UI to be secondary to core gameplay, and therefore depend on the specifics of the game. However, the upside (and I’m sure the point) is this approach forces the various menus and UIs to implement general interfaces rather than being specific to one particular implementation of the game objects.

Something that came up on Friday that was somewhat new for me was the concept of static fields. This came up because, one of the stories requires that the game setup menu provide players with a default identifier choice. However, this default choice had to be randomly chosen from a list of names. To do this I ended up creating a helper class that selects a random element from a list based on the SO answer referenced below. However, the author suggested not using multiple instances of the `Random` class and instead using a static field. The `Random` docs warn against using multiple instances, since the empty constructor uses the system clock for the seed, so instantiating two Random objects in quick succession could result in them generating the same sequences.

This all made sense aside from the part regarding using a static field as static fields weren’t something that I had used before. But after looking at the C# docs I better understood why the author was suggesting using a static field in this case. 

> “Only one copy of a static member exists, regardless of how many instances of the class are created” 

The really simple example below shows how this works.

``` csharp
   public class StaticMemberExample
   {
       public static int InstanceCount { get; private set; }

       public StaticMemberExample()
       {
           InstanceCount += 1;
       }

       static void Main(string[] args)
       {
           Console.WriteLine(string.Format("There are {0} of me", StaticMemberExample.InstanceCount));
           var sme1 = new StaticMemberExample();
           Console.WriteLine(string.Format("There are {0} of me", StaticMemberExample.InstanceCount));
//          Cannot access with instance reference
//          Console.WriteLine(sme1.InstanceCount);
           var sme2 = new StaticMemberExample();
           Console.WriteLine(string.Format("There are {0} of me", StaticMemberExample.InstanceCount));
           var sme3 = new StaticMemberExample();
           Console.WriteLine(string.Format("There are {0} of me", StaticMemberExample.InstanceCount));
//            Output:
//            There are 0 of me
//            There are 1 of me
//            There are 2 of me
//            There are 3 of me
       }
   }

```

So to avoid two identical sequences the author was suggesting all instances essentially share the same random sequence. 

*** This approach is not thread safe. See the two posts by Jon Skeet below for details. 

### Resources:

[SO: Access Random Item in List](http://stackoverflow.com/questions/2019417/access-random-item-in-list)

[Static Classes and Static Class Members (C# Programming Guide)](https://msdn.microsoft.com/en-us/library/79b3xss3.aspx)

[C# Random](https://msdn.microsoft.com/en-us/library/ctssatww(v=vs.110).aspx)

[Jon Skeet: Static Random](http://www.yoda.arachsys.com/csharp/miscutil/usage/staticrandom.html)

[Jon Skeet: Revisiting Randomnees](https://codeblog.jonskeet.uk/2009/11/04/revisiting-randomness/)


