---
author: alc
category:
  - item-extensions
  - link-database
  - sitecore
  - sitecore-basics
date: "2013-10-13T20:19:11+00:00"
guid: http://laubplusco.net/?p=116
title: Get referrers as Sitecore items through the link database
url: /sitecore-item-extensions-get-referrers-as-items/

---
_This is my first post in a series about creating .NET extension methods on a Sitecore item. I love extension methods and have been using them to wrap common operations on Sitecore items since they were introduced in .NET 3. The series of posts will also contain an architectural description on how to implement and maintain your extension methods so these are re-usable in all of your Sitecore solution_ s.

Instead of starting with the architectural thoughts I will dive deep into coding one of my favorite extension methods.

## Get referrers on an item

Okay, what we want to achieve with this item extension is to get all items which points to theitem without using a Lucene index, instead we use the Sitecore link database.

```c#
public static Item[] GetReferrersAsItems(this Item item, bool includeStandardValues = false)
{
  var links = Globals.LinkDatabase.GetReferrers(item);
  if (links == null)
    return new Item[0];
  var linkedItems = links.Select(i => i.GetSourceItem()).Where(i => i != null);
  if (!includeStandardValues)
    linkedItems = linkedItems.Where(i => !i.Name.Equals("__standard values", StringComparison.InvariantCultureIgnoreCase));
  return linkedItems.ToArray();
}
```

Using the link database is really easy as this code shows. To retrieve the referrers simply call the Globals.LinkDatabase.GetReferrers with the item passed as a parameter.

This returns a collection of ItemLink which contain a source and a target item id, database, etc. along with convenient methods for getting the source or target item. Calling these methods for each item will result in a linear amount of item fetches compared with retrieving the ItemLinks collection which is one database call to the link database.

Getting all referred items can thus be quite heavy.

The returned ItemLink collection will also contain standard values, branches and so on, not only content from within a site. To filter out the standard values a simple name comparison check is made to filter this out before returning the items.

## Get referrers only within the context site

Well what if we only want the items from within the current context site. This can be done simply by determining if an item resides beneath the current sitecontext start path.

```c#
public static Item[] GetReferencesInContextSite(this Item item)
{
  var siteContext = Context.Site;
  if (siteContext == null)
    return new Item[0];
  var referrers = item.GetReferencesAsItems());
  return referrers.Where(i => i.Paths.FullPath.StartsWith(siteContext.StartPath, StringComparison.InvariantCultureIgnoreCase)).ToArray();
}
```

Please note that this is not always a good assumption to make and might fail miserably on index/taxonomy based solutions.

The way Sitecore is moving also means that the paths "sitecore/content", "sitecore/layouts" and "sitecore/templates" no longer have to exist in a solution as fixed paths. Template items, layouts and so on can potentially live anywhere in the future.

A warning

Using these methods can be extremely heavy. For example if getting all referrers of a template which is used in a bucket with millions of items. This will take minutes or more and will slow up your site.

Using this method requires discipline and should only be used for example retrieving settings items from a template where only one per site or maybe only one per solution exists. An example on how to do this will come in a later blog post.

It can also be used to get all items which refers to the one being published and then publish these depending on some conditions not to start a publish nightmare.

I dare you to like this post..
