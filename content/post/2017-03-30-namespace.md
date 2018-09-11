---
categories: 
  - apprenticeship
date: "2017-03-30T13:30:00Z"
title: namespace
---

Every time you create a new file, Rider automatically adds a namespace declaration that corresponds to the directory that the new file is located (`namespace Directory.SubDirectory`). At first I just accepted this, but eventually I started to wonder what the heck is a namespace? 

In .NET namespaces are used to organize multiple related objects and help to avoid collisions between objects sharing the same name. They don’t necessarily correspond to the directory structure of the project even though this seems to be the recommended approach. Overall, it seems like namespaces give you a bit more flexibility in how you organize your project (as I’m used to modules having to mimic the directory structure of the project), even though generally namespaces seem to just follow the project’s directory structure. 

If you are going to continually be using a namespace you can add a `using` declaration at the top of the file, which allows you to avoid typing the fully qualified name. Or you can alias longer namespaces if you’d still prefer to have something closer to a fully qualified reference throughout the code. 

#### Fully Qualified Name
``` csharp
using System;

namespace TopLevel
{
   namespace MidLevel
   {
       namespace BottomLevel
       {
           public class MyClass
           {
               public string Value = "HI";
           }
       }
   }
}

public class NameSpaceExample
{
   public void Main(string[] args)
   {
       var myClass1 = new TopLevel.MidLevel.BottomLevel.MyClass();
       Console.WriteLine(myClass1);
   }
}
```

#### `using` directive
``` csharp
using System;
using TopLevel.MidLevel.BottomLevel;

namespace TopLevel
{
   namespace MidLevel
   {
       namespace BottomLevel
       {
           public class MyClass
           {
               public string Value = "HI";
           }
       }
   }
}

public class NameSpaceExample
{
   public void Main(string[] args)
   {
       var myClass1 = new MyClass();
       Console.WriteLine(myClass1);
   }
}
```
#### Alias
``` csharp
using System;
// Can alias
using tmb = TopLevel.MidLevel.BottomLevel;

namespace TopLevel
{
   namespace MidLevel
   {
       namespace BottomLevel
       {
           public class MyClass
           {
               public string Value = "HI";
           }
       }
   }
}

public class NameSpaceExample
{
   public void Main(string[] args)
   {
       var myClass = new tmb.MyClass();
       Console.WriteLine(myClass);
   }
}

```

### Resources:
[https://msdn.microsoft.com/en-us/library/z2kcy19k.aspx](https://msdn.microsoft.com/en-us/library/z2kcy19k.aspx)

[http://stackoverflow.com/questions/4664/should-the-folders-in-a-solution-match-the-namespace](http://stackoverflow.com/questions/4664/should-the-folders-in-a-solution-match-the-namespace)



