---
author: alc
category:
  - pipelines
  - sitecore
date: "2013-10-15T19:56:54+00:00"
guid: http://laubplusco.net/?p=153
title: Inheriting renderings in Sitecore
url: /inheriting-renderings-sitecore/

---
A typical page layout contains an aside column beside the main content.

These aside columns typically contain spots, boxes or whatever you will call them.

[![asidespot](/wp-content/uploads/2013/10/asidespot-1024x609.png)](/wp-content/uploads/2013/10/asidespot.png)

We call them spots at Pentia.

What is the problem then? When using the page editor for inserting and personalizing renderings then there is a 1 - 1 relation with the rendering reference that is inserted and the item.

Inserting aside spots on each and every page is extremely time consuming for editors and might also exceed their Sitecore knowledge if they're not trained.

What if the spot renderings simply could be inherited from the parent item? So it is not necessary to put a spot on a sub-sub-sub-sub page but still keep the aside column filled with dynamic content? This was easy before the page editor and personalization became the Sitecore way, back then it was almost just inheriting a field value but today it is a bit more complicated.

The aside column is just an example. The same inheritance mechanism shown below can also be applicable on other placeholders in which you would like renderings to be inherited from the parent or another ancestor of the current item.

## Inheriting renderings from ancestors

First we create a new user control (or rendering when using MVC) containing a placeholder for our aside spots.

```xhtml
<%@ Control Language="C#" AutoEventWireup="true" CodeBehind="AsideSpotsHolder.ascx.cs" Inherits="[NAMESPACE].AsideSpotsHolder" %>
<%@ Register TagPrefix="sc" Namespace="Sitecore.Web.UI.WebControls" Assembly="Sitecore.Kernel" %>
<sc:Placeholder runat="server" ID="AsideSpots" Key="AsideSpots" />
```

Then we register the sublayout/rendering in Sitecore and save the ID in an Constants file.

```c#
  internal class Constants
  {
    #region Nested type: Keys

    internal struct Keys
    {
      internal const string AsideSpotsKey = "AsideSpots";
    }

    #endregion

    #region Nested type: Renderings

    internal struct Renderings
    {
      internal const string AsideSpotsHolderId = "{6D0761A9-6506-4D80-B9C5-09017D3F8231}";
    }

    #endregion
  }
```

In this example we just use a "hardcoded" placeholder key in the Constants file called "Constants.Keys.AsideSpotsKey" for the placeholder key in which we want to inherit renderings from the parent.

Next step is to create a processor for the insertRenderings pipeline.

```c#
public class AddFallbackRenderings : InsertRenderingsProcessor
{
  public override void Process(InsertRenderingsArgs args)
  {
    Assert.ArgumentNotNull(args, "args");
    if (!args.HasRenderings || args.ContextItem == null || Context.Site.DisplayMode == DisplayMode.Edit || InheritRenderingsService.SkipInherit(args.ContextItem))
      return;

    using (new ProfileSection("Inserting inherited renderings"))
    {
      if (!args.Renderings.Any(r => r.Placeholder.EndsWith(GetPlaceHolderKey(Constants.Keys.AsideSpotsKey), StringComparison.InvariantCultureIgnoreCase)))
        InheritRenderingsService.Inherit(args.ContextItem.Parent, args.Renderings, new ID(Constants.Renderings.AsideSpotsHolderId), Constants.Keys.AsideSpotsKey);
    }
  }

  protected virtual string GetPlaceHolderKey(string splotplaceholderKey)
  {
    return StringUtil.EnsurePrefix('/', splotplaceholderKey);
  }
}
```

A bit of explanation. First we check if the current item has renderings at all (see this [post](/item-extensions-item-has-renderings/ "Does an item have renderings")), then we check if we are in Edit mode (the page editor) and then we check if inheritance is skipped on the item, more on this later in the post.

If the checks does not exit the processor then a check is made if the item already has renderings in a placeholder with that key.

If the item does not have any renderings in the placeholder already then we call a service class with the item parent as argument along with the rendering references already found in the insertRenderings pipeline, the sublayout ID and the placeholder key.

```c#
internal class InheritRenderingsService
{
  internal static void Inherit(Item inheritItem, List<RenderingReference> targetRenderings, ID renderingId, string placeholderKey)
  {
    if (inheritItem == null || SkipInherit(inheritItem) || inheritItem.Paths.FullPath.Equals(Context.Site.StartPath, StringComparison.InvariantCultureIgnoreCase))
      return;

    var renderings = inheritItem.Visualization.GetRenderings(Context.Device, true);
    if (!renderings.Any(r => r.RenderingID.Equals(renderingId)))
      return;
    var renderingsToInherit = renderings.Where(r => r.Placeholder.EndsWith(placeholderKey, StringComparison.InvariantCultureIgnoreCase)
      || r.Placeholder.ToLowerInvariant().Contains(string.Concat(placeholderKey, "/").ToLowerInvariant())).ToArray();
    if (!renderingsToInherit.Any())
      Inherit(inheritItem.Parent, targetRenderings, renderingId, placeholderKey);
    else
      InsertRenderings(renderingsToInherit, targetRenderings);
  }

  private static void InsertRenderings(RenderingReference[] renderingsToInherit, List<RenderingReference> targetRenderings)
  {
    foreach (var renderingReference in renderingsToInherit)
    {
      renderingReference.Placeholder = CleanPlaceholderName(renderingReference.Placeholder);
    }
    targetRenderings.InsertRange(targetRenderings.Count, renderingsToInherit);
  }

  private static string CleanPlaceholderName(string placeholderKey)
  {
    var placeholderKeys = placeholderKey.Split('/');
    return placeholderKeys[placeholderKeys.Length - 1];
  }

  internal static bool SkipInherit(Item item)
  {
    var checkBox = new CheckboxField(item.Fields[Constants.Fields.StopSpotInheritance]);
    return checkBox == null || checkBox.Checked;
  }
}
```

The Inherit method is called recursively until a condition fails or until some renderings to inherit is found. These conditions are if the item to inherit from is null or if it is the start item of the site. It is also checked if the inheritance should be skipped on that item as also mentioned above.

If renderings are found on the item to inherit from then they  are inserted into the current items rendering collection which was passed as a parameter to the Inherit method.

Now to making the SkipInherit part, it is only a checkbox so should be fast.

Make a base class in Sitecore and let all pages where rendering inheritance should be enabled inherit from that.

[![spotinheritance](/wp-content/uploads/2013/10/spotinheritance.png)](/wp-content/uploads/2013/10/spotinheritance.png)

Last but not least we need to place the AddFallbackRenderings processor in the insertRenderings pipeline. To make it work we need to patch the processor just before  Sitecore.Pipelines.InsertRenderings.Processors.EvaluateConditions

```xhtml
<processor type="[NAMESPACE].AddFallbackRenderings, [ASSEMBLY]" />
```

That is it, renderings are inherited.

This code could easily be generalized and made into a module which read the placeholder keys and rendering ID's from a settings item. The use of static methods and internals should be revised then to make it flexible and extendible.

Another approach could be to make a set of actions and conditions for the rules engine. I am working a bit on this approach so a blog post will follow on this some time soon.

Let me know if you like the rules idea and want to contribute or if you like me to share the source code from this post, just write a comment.
