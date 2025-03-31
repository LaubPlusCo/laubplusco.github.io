---
author: alc
category:
  - item-extensions
  - languages
  - sitecore
  - sitecore-basics
date: "2013-10-15T17:51:46+00:00"
guid: http://laubplusco.net/?p=143
title: Sitecore item languages
url: /sitecore-extensions-sitecore-item-languages-item-extension/

---
_This is my fourth post in a series about creating .NET extension methods on a Sitecore item. I love extension methods and have been using them to wrap common operations on Sitecore items since they were introduced in .NET 3. The series of posts will also contain an architectural description on how to implement and maintain your extension methods so these are re-usable in all of your Sitecore solution_ s.

Okay this is going to be a quick one about useful extension methods on the Item class when working on a multilingual site. There are no revolutions in this code and the methods are probably duplicates from a ton of other blogs. They are useful nonetheless so they deserve a post in my extension methods series.

## Sitecore item languages

This small one checks if an item has any version on the supplied language.

```c#
public static bool HasLanguage(this Item item, Language language)
{
  return ItemManager.GetVersions(item, language).Count > 0;
}
```

And here is an overload which takes the language name as a string.

```c#
public static bool HasLanguage(this Item item, string languageName)
{
  return item.HasLanguage(LanguageManager.GetLanguage(languageName, item.Database));
}
```

## Context language

Here is one for checking if the item has a version on the context language. Instead of checking if it have any versions on the language, it simply checks if the item has a version at all.

```c#
public static bool HasContextLanguage(this Item item)
{
  if (item == null || item.Versions == null || item.Versions.Count == 0)
    return false;
  var latestLanguageVersion = item.Versions.GetLatestVersion();
  return (latestLanguageVersion != null) && (latestLanguageVersion.Versions.Count > 0);
}
```

That's it. In a later post I will show how a guaranteed language fallback mechanism can be implemented using these methods as support.

I dare you to like this post..
