---
author: alc
category:
  - item-extensions
  - link-database
  - sitecore
  - sitecore-basics
date: "2013-10-13T20:33:49+00:00"
guid: http://laubplusco.net/?p=121
title: Has references to a Sitecore item
url: /sitecore-item-extensions-has-references-to-an-item/

---
_This is my second post in a series about creating .NET extension methods on a Sitecore item. I love extension methods and have been using them to wrap common operations on Sitecore items since they were introduced in .NET 3. The series of posts will also contain an architectural description on how to implement and maintain your extension methods so these are re-usable in all of your Sitecore solution_ s.

Following my last post about getting referrers as items I want to follow up with two other kind of cute extension methods which use the link database.

## IsReferencedByItem

This small extension method simply checks if an item is referenced by another item ID using the link database.

```c#
public static bool IsReferencedByItem(this Item item, ID referrerItemId)
{
  var links = Globals.LinkDatabase.GetReferrers(item);
  return links != null && links.Any(l => l.SourceItemID == referrerItemId);
}
```

For convenience I also made an overload which takes an item as parameter instead of the ID.

```c#
public static bool IsReferencedByItem(this Item item, Item referrerItem)
{
  return referrerItem != null && item.IsReferencedByItem(referrerItem.ID);
}
```

## HasReferenceToItem

And then the other way around. This extension method checks if an item references an item ID using the link database.

```c#
public static bool HasReferenceToItem(this Item item, ID referencedItemId)
{
  var links = Globals.LinkDatabase.GetReferences(item);
  return links != null && links.Any(l => l.TargetItemID == referencedItemId);
}
```

And also for convenience I made an overload which takes the item as a parameter instead of the ID.

```c#
public static bool HasReferenceToItem(this Item item, Item referencedItem)
{
  return referencedItem != null && item.HasReferenceToItem(referencedItem.ID);
}
```

That was it.

I dare you to like this post..
