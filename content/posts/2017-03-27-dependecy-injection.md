---
layout: post
title:  Dependency Injection
date:   2017-03-27 16:46:00 -0500
categories: 
  - apprenticeship
---

On Friday my mentors and I had our first planning meeting. It was interesting to go through the process of mapping out very specific acceptance criteria along with estimating each story. Most of the new stories that we created centered around building out a series of menus that collect the player’s preferences before the TTT game starts. The pre-apprenticeship submission of TTT had a requirement for a similar set of pre-game menus. I think designing and testing the pre-game menus was probably the part of my submission that took me the most time, so it will be good to take another crack at it.

Today, I mainly worked on completing the first of the assigned stories, which was to build out a simple menu to allow players to choose the grid size of their TTT game. It was pretty straightforward to do this sort of thing, but I wasn’t quite sure how to test the correctness of the various messages being read from and written to the console. After a bit of research, it seemed like using dependency injection was a good solution to this type of problem. Wikipedia defines dependency injection as follows:

> “In software engineering, dependency injection is a technique whereby one object supplies the dependencies of another object. A dependency is an object that can be used (a service). An injection is the passing of a dependency to a dependent object (a client) that would use it. The service is made part of the client's state.[1] Passing the service to the client, rather than allowing a client to build or find the service, is the fundamental requirement of the pattern.”

So rather than initializing a containing class’s dependencies inside of the class they are instead initialized outside of the containing class and passed in via the constructor or a setter. There are [several advantages to using this approach approach](https://en.wikipedia.org/wiki/Dependency_injection#Advantages), but the immediate benefit in my situation was that it was now much easier to test my `StartUpMenu` class.

Originally, my `StartUpMenu` class had directly depended on the console. One way remove this dependency is to inject a class or interface at construction of the `StartUpMenu` class. The modified version of my StatupMenu class requires that it be initialized with TextReader and TextWriter objects. These are both general stream processing classes (which Console.In and Console.Out are derived from). This made it easier to test the `StartUpMenu` class, since I could now directly feed in the test input to mimic a user interacting with the Console. 

### Original StartUpMenu
``` csharp
public class StartUpMenu
{
   private readonly List<SubMenu> _menus;

   public StartUpMenu(List<SubMenu> menus)
   {
       _menus = menus;
   }
...
   public string ValidateInput(SubMenu menu)
   {
       string input;
       bool valid;
       do
       {
            // Depends on Console
           Console.WriteLine(menu.Message);
          // Depends on Console
           input = Console.ReadLine().Trim();
           valid = menu.IsValidInput(input);
           if (!valid)
           {
	  // Depends on Console
               Console.WriteLine(menu.ErrorMessage);
           }
       } while (!valid);
       return input;
   }
...
}

```
### Updated StartUpMenu
``` csharp
public class StartUpMenu
{
   private readonly TextReader _textReader;
   private readonly TextWriter _textWriter;
   private readonly List<SubMenu> _menus;

   public StartUpMenu(TextReader textReader, TextWriter textWriter, List<SubMenu> menus)
   {
       _textReader = textReader;
       _textWriter = textWriter;
       _menus = menus;
   }
  
...   
   public string ValidateInput(SubMenu menu)
   {
       string input;
       bool valid;
       do
       {
            // No longer depends on Console
           _textWriter.WriteLine(menu.Message);
            // No longer depends on Console
           input = _textReader.ReadLine().Trim();
           valid = menu.IsValidInput(input);
           if (!valid)
           {
	   // No longer depends on Console
               _textWriter.WriteLine(menu.ErrorMessage);
           }
       } while (!valid);
       return input;
   }
...
}

```


## Resources:
[Wiki: Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection)

[SO: Thread with a good discussion](http://stackoverflow.com/questions/14301389/why-does-one-use-dependency-injection)




