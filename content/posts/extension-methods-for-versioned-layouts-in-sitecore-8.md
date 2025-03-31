---
author: alc
category:
  - item-extensions
  - sitecore
  - sitecore-8
cover:
  alt: html_for_babies
  image: /wp-content/uploads/2015/01/html_for_babies.png
date: "2015-01-11T21:27:29+00:00"
guid: http://laubplusco.net/?p=1298
title: Extension methods for versioned layouts in Sitecore 8
url: /extension-methods-for-versioned-layouts-in-sitecore-8/

---
Finally I am back blogging. The last 1½ month of my paternity leave and the first week of work did not go exactly as I had planned. Looking on the bright side I have learned a lot about how a baby's immune system evolves. It goes something like this, baby gets a virus/bacteria, it incubates and when it is strong enough it attacks the parents (the virus, not the baby). One virus or bacteria rarely visit a baby alone since their immune system is weakened making them easy prey. This means a common cold suddenly can become severe diarrhea (feasis everywhere) and then pneumonia that needs treatment with penicillin which then can feed some mushroom loving rectal e. coli bacteria that causes diarrhea and so on..

In 2-3 years time he has probably been through all the common stuff and then his immune system will be strong. The saying "What doesn't kill you ... " fits perfectly for babies.

> Enough about babies for now..

For a long time I've used these extension methods on a Sitecore item for checking if the item has a layout defined.

```c#
public static bool HasRenderings(this Item item)
{
  return item.Visualization.GetRenderings(Context.Device, false).Any();
}

public static bool HasRenderings(this Item item, DeviceItem device)
{
  return item.Visualization.GetRenderings(device, false).Any();
}
```

It simply checks if there are any renderings defined for a specific device.

This extension method was fine back when the world was a simple place and it still works fine to determine if there are any layout at all set on an item. Calling Item.Visualization.GetRenderings(..) still works and returns 0 renderings if there are none in Sitecore 8 as well.

## Versioned layouts in Sitecore 8

Sitecore 8 introduce the concept of versioned layouts. This has been a long time request from the community both to support different delta renderings on different languages but also to differ layouts between different versions of an item .

The solution that Sitecore has come up with is an introduction of a new versioned and unshared field called _\_\_Final Renderings._ The Sitecore documentation can be found [here](https://doc.sitecore.net/products/sitecore%20experience%20platform/versioned%20layouts/versioned%20layouts).

Since the original \_\_Renderings field contains Sitecore's own layout xml I am considering if adding support for an optional language and version attribute in the xml would have been sufficient to cover the needs that the community requested. This could potentially have been a simpler solution than adding a new field. Now the two field values are patched together which might seem confusing to some.

When you add any rendering through the Experience Editor (the UI formerly known as the Page Editor), the rendering will be added to the \_\_Final Renderings field thus be versioned by default.

This means that if the editors rely on the Experience Editor and have multiple languages on a site then they need to edit the pages on all languages and insert all renderings instead of having the option simply to translate the datasources.

It would have been nice to give the editor the option to choose if a rendering that is inserted via the Experience Editor should be versioned or shared. It could simply be a checkbox that picks up a default value set on the rendering item reducing the workload for editors. The functionality is more or less already there. The checkbox should simply switch between storing the delta rendering in the \_\_Renderings or the \_\_Final Renderings field depending on if you want it shared or not.

Sitecore has added an extra tab in the Presentation Details dialog to show the shared and the "final" (versioned) layout details. The Presentation Details dialog is accessible from the Advanced strip in the Experience Editor ribbon.

![layout_details_button_sitecore_8](/wp-content/uploads/2015/01/layout_details_button_sitecore_8.png)

They also extended the Reset Layout dialog so that it now can reset both the "Final" (versioned) and the shared layout.

![reset_versioned_layout_sitecore_8](/wp-content/uploads/2015/01/reset_versioned_layout_sitecore_8.png)

Some editors will probably not be happy at all when they realize that they cannot use the Experience Editor UI to build up all the layout of their pages on all languages and then just translate the content items as they have been used to do.

Other editors will be extremely happy that they now can have a single website in many languages that differ in layout between languages.

It seems a bit like going from one extreme to the other. In the previous versions of Sitecore all layouts was shared between languages when using the page editor and now nothing is shared when using the new Experience Editor.

The Presentation details has been made easier to access but is still more advanced to use than the Experience Editor which is why it is also placed under Advanced. From a Sitecore developer perspective we seldom want any editor to fiddle around in the presentation details dialog.

Well now this is the approach that Sitecore chose to follow to give us versioned layouts and I am sure that they've had good reasons for choosing the solution.

Whatever solution they had gone with there would have been ups and downs. It is great to have versioned layouts now and some retrofitting is of course required when such a fundamental change is introduced.

Checkboxes in the Experience Editor for selecting if renderings are versioned or not would be appreciated. A "Copy layout to other language" button could also be nice to have out of the box. We just need something that will please the customers who's requirements fitted with the shared layouts. Versioned layouts are really really great and has been missed but they are not the right solution for everyone.

Now to the code..

_**We need some extra extension methods though to check if an item has versioned renderings.**_

This one checks if an item has versioned renderings:

```c#
public static bool HasVersionedRenderings(this Item item)
{
  if (item.Fields[FieldIDs.FinalLayoutField] == null)
    return false;
  var field = item.Fields[FieldIDs.FinalLayoutField];
  return !string.IsNullOrWhiteSpace(field.GetValue(false, false));
}
```

And these ones can check if an item has versioned renderings on the latest version of a specific language, any language and a convenience one for testing the context language:

```c#
public static bool HasVersionedRenderingsOnLanguage(this Item item, Language language)
{
  return item != null && item.Database.GetItem(item.ID, language).HasVersionedRenderings();
}

public static bool HasVersionedRenderingsOnAnyLanguage(this Item item)
{
  return ItemManager.GetContentLanguages(item).Any(item.HasVersionedRenderingsOnLanguage),
}

public static bool HasVersionedRenderingsOnContextLanguage(this Item item)
{
  return item.HasVersionedRenderingsOnLanguage(Context.Language);
}
```

This one checks if an item has versioned renderings on a specific version:

```c#
public static bool HasVersionedRenderingsOnVersion(this Item item, Language language, Sitecore.Data.Version version)
{
  var versionItem = item.Database.GetItem(item.ID, language, version);
  return versionItem != null && versionItem.HasVersionedRenderings();
}
```

_**Warning!** These extension methods rely on the static field ID FieldIDs.FinalLayoutField from the Sitecore 8 kernel which means that these will not work on an older Sitecore solution._

If you want backwards compatibility in your extension method library then simply replace FieldIDs.FinalLayoutField with new ID("{04BF00DB-F5FB-41F7-8AB7-22408372A981}").

I have not worked so much with versioned layouts yet so I will probably write more on this in the future.

Please feel free to share your thoughts on the new versioned layouts in the comments below.
