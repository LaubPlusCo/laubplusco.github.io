---
author: alc
category:
  - item-extensions
  - sitecore
  - sitecore-basics
date: "2013-11-07T21:55:33+00:00"
guid: http://laubplusco.net/?p=281
title: Does a Sitecore item derive from a template
url: /sitecore-extensions-does-a-sitecore-item-derive-from-a-template/

---
_This is my fifth post in a series about creating Sitecore .NET extension methods on a Sitecore item. I love extension methods and have been using them to wrap common operations on Sitecore items since they were introduced in .NET 3. The series of posts will also contain an architectural description on how to implement and maintain your extension methods so these are re-usable in all of your Sitecore solution_ s.

This is probably one of the most common Sitecore extension methods ever made.

During my work I've made a number of health checks on other Sitecore partner solutions and this method has in some form shown up in almost all of them. In most cases they have been splattered with some domain specific logic and mile long code statements but the primary purpose always remain the same.That is to check if an item is created from a template which either is or derives from the provided template.

At Pentia the method is always called "IsDerived" and kept well out of reach of anything domain specific in a separate module. More about extension method architecture in a soon-to-come post.

The name of the method could also be IsBasedOnTemplate(ID templateId) or maybe even just Is(ID templateId).

The method is so common that it is almost weird it is not already a standard method on an item. This is why it deserves a blog post even though most Sitecore developers knows the concept by heart.

## Checking if an item derives from a template

First we create an extension method on the Template class which checks if a Template has the provided ID or is based on a template with the ID recursive looking through all base templates.

```c#
  public static class TemplateExtensions
  {
    public static bool IsDerived([NotNull] this Template template, [NotNull] ID templateId)
    {
      return template.ID == templateId || template.GetBaseTemplates().Any(baseTemplate => IsDerived(baseTemplate, templateId));
    }
  }
```

Then we create an extension method for the Item class which takes an ID as a parameter.

```c#
public static class ItemExtensions
{
  public static bool IsDerived([NotNull] this Item item, [NotNull] ID templateId)
  {
    return TemplateManager.GetTemplate(item).IsDerived(templateId);
  }
}
```

We can then also make a convenience extension methods to get all children deriving from etc.

```c#
public static IEnumerable<Item> GetChildrenDerivedFrom(this Item item, ID templateId)
{
  return item.GetChildren().Where(c => c.IsDerived(templateId));
}
```

 _Please note that getting all children of an item is not always a good idea, think ahead before iterating over a childcollection also more than just 5 minutes into the future._

That was it for this post, soon to come a post about where to place all of these extension methods.
