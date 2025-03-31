---
author: alc
category:
  - item-extensions
  - sitecore
  - sitecore-basics
date: "2013-10-13T20:46:18+00:00"
guid: http://laubplusco.net/?p=126
title: Does an item have renderings
url: /sitecore-item-extensions-item-has-renderings/

---
_This is my third post in a series about creating .NET extension methods on a Sitecore item. I love extension methods and have been using them to wrap common operations on Sitecore items since they were introduced in .NET 3. The series of posts will also contain an architectural description on how to implement and maintain your extension methods so these are re-usable in all of your Sitecore solution_ s.

This is yet another quick extension method which has proven to be really useful. I will re-use it in later posts so better write about it first.

## HasRenderings

The extension method checks if an item has renderings for the device item which is passed as parameter.

```c#
public static bool HasRenderings(this Item item, DeviceItem device)
{
  return item.Visualization.GetRenderings(device, false).Any();
}
```

As an overload I made a version which passes the Context.Device to the method as deviceItem parameter.

```c#
public static bool HasRenderings(this Item item)
{
  return item.HasRenderings(Context.Device);
}
```

That was it.

I dare you to like this post..
