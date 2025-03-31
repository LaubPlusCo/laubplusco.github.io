---
author: alc
category:
  - best-practices
  - sitecore
cover:
  alt: nested hell
  image: /wp-content/uploads/2014/01/nestedhell.png
date: "2014-01-12T22:24:00+00:00"
guid: http://laubplusco.net/?p=589
title: Nesting if statements
url: /nesting-statements/

---
This post is not only about Sitecore development it is valid for any code written in almost any language.

It is about making code easier to read. The post is somewhat subjective but I hope that all developers with an IQ higher than 80 will agree.

Back in the days when I learned to write code I could end up writing something  like this:

```c#
      public void SomeMethod(string args)
      {
        if (args != null)
          var something = args.Something;
          if (something != null)
          {
            var someResult = DoSomethingWithSomething(something);
              if (someResult != null)
              {
                IGotMyResult(someResult);
              }
          }
      }
```

But what I was doing back then was actually turning things upside down, writing "stay with me" instead of saying _**no**_ right away, making the code less readable for other developers.

Today I would always write the same piece of code like this:

```c#
      public void SomeMethod(string args)
      {
        if (args == null)
          return;
        var something = args.Something;
        if (something == null)
          return;
        var someResult = DoSomethingWithSomething(something);
        if (someResult == null)
          return;
        IGotMyResult(someResult);
      }
```

The point being, STOP nesting if statements.

If a condition is not met then exit right away, don't write "if condition met then do something" rather write "if condition not met then return" -  just exit, otherwise your method has to much responsibility.

This is kind of a continuation of my post about exiting fast in pipeline processors which I wrote some months ago [here](/sitecore-pipelines/ "A note about Sitecore pipelines").

I hope all readers of this post will listen and change their approach if they are still nesting if statements. It is just an unnecessary waste of white spaces.
