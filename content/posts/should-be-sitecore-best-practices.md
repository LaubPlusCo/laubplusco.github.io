---
author: alc
category:
  - best-practices
  - sitecore
  - sitecore-basics
cover:
  alt: bestpractice
  image: /wp-content/uploads/2014/01/bestpractice.png
date: "2014-01-06T20:22:03+00:00"
guid: http://laubplusco.net/?p=512
title: Should be Sitecore best practices
url: /should-be-sitecore-best-practices/

---
During my work at [Pentia](http://www.pentia.dk "Pentia A/S") I often perform health checks of existing Sitecore solutions built by other Sitecore partners.

The solutions vary a lot in both quality and age but there are a few things which I need to point out as missing in each and every report.

It is like it is secret Sitecore features only known by a small group of privileged developers even though it is so extremely basic.

The customer who receive the health check will not have  so much focus on the code quality (even though this is where I have my fun) their focus is often more on the usability for their editors.

So the first thing I check when I have access to the Sitecore back-end is how the solution templates are structured and how fields are named etc. and the three points listed below almost always come up.

## 1\. Always set an icon on template sections

All standard template sections in Sitecore have a specific icon shown in the content editor so the section is easy to identify for developers:

[![Standard Template Section icons](/wp-content/uploads/2014/01/StandardTemplateSections.png)](/wp-content/uploads/2014/01/StandardTemplateSections.png)

When you create a new section it will just have this "dummy" icon from the default Data section item which we all know:

[![Standard Data Section Icon](/wp-content/uploads/2014/01/StandardDataSectionIcon.png)](/wp-content/uploads/2014/01/StandardDataSectionIcon.png)

To help the editors identify different sections on an item then provide the section with a unique icon.

To change the section icon simply change the icon of the section item created beneath the template item:

[![Setting an Icon on a Section](/wp-content/uploads/2014/01/SettingAnIconOnASection.png)](/wp-content/uploads/2014/01/SettingAnIconOnASection.png)

This is almost always skipped on most or even all solution specific template sections even though it takes less than a minute for a developer to set up and really improves the daily work for the editors (not alone their first impression of the solution).

## 2\. Always set a Display Name on template sections

When using the template builder the "default" section name has been "Data" since forever.

[![The Data Section](/wp-content/uploads/2014/01/TheDataSection.png)](/wp-content/uploads/2014/01/TheDataSection.png)

I truly hate the section name "Data" by now. It says absolutely nothing about the fields and which type of data they contain.

So first thing is, always call your sections something which is related to the fields which the section contains.

Next thing is to translate this into a nice name for the editors. To do this set the display name of the item to a translated name which makes sense:

[![Section display name](/wp-content/uploads/2014/01/SectionDisplayName.png)](/wp-content/uploads/2014/01/SectionDisplayName.png)

What if the editors only speak English from now on and forever? Then still please give the section a nice display name in English. The example above would then be item name _SearchResultsPage_ while the display name would be _Search Results Page_ with spaces.

## 3\. Always set a Title on Fields

My last hassle for today is field names. It seems like a lot of Sitecore developers either have forgotten about the Title field on Field items or do not know that it exists.

To set a nice title maybe even with a short explanation for the editor then use the Title field on the field item:

[![Set field title](/wp-content/uploads/2014/01/SetFieldTitle1.png)](/wp-content/uploads/2014/01/SetFieldTitle1.png)

I often come across a field name which contain spaces where the developer have written a whole sentence as a name or where the field name is prefixed with something that might have made sense for the developer but for me as an outsider is complete nonsense:

[![Bad bad field names](/wp-content/uploads/2014/01/BadBadFieldNames.png)](/wp-content/uploads/2014/01/BadBadFieldNames.png)

(f, what the f does that mean?)

I feel very bad for the editors who have had to work with this compared with how nice it could have been if the developer would have spend just 2-3 minutes time considering what the fields should be called and then set some nice titles on the field items afterwards. So little effort but so much is gained.

Using the Title field also allow you as an editor to write field names which are easy to use in code. (I would still recommend _always_ to use the ID of fields in code instead of the name since there is no guarantee that a field name is unique on an item and ID's are supposedly a very tiny tiny bit faster, place ID's in a Constants file. See the Sitecore class FieldIDs for inspiration).

## Summary

- Always set icons on template sections.
- Always give your template sections a name which says something about that section of fields (Do not call it Data except if it makes total sense for the editors).
- Always set a nice display name on template section items on all editor languages
- Never try to write field names which explains the field in a sentence
- Never pre- or postfix field names with other than the section name
- Always set the title field on the field item, use this for explaining the field

These very basic things are almost always either completely or partly missing on all solutions. It is a real shame since it is so easy to implement.

Some might argue _"But Sitecore uses the default Data section as well for many of their system items"._

Well yes that is true,  see for example the Data section on a Field item which contains the Title field shown above. But this is for developers and not for editors. It would have been nice of Sitecore though if they had retained the consistency in all system sections as they have in the standard sections.

A lot of other basic stuff are typically missing as well such as benefiting from template inheritance etc. so I will probably write another post on this soon when I have some time.

Last, don't worry if you missed some of these things many times or maybe didn't even know how to set icons on sections. It is not shown at Sitecore developer training or at least not back when I was certified some 8 years ago. It should be since it is so easy and greatly improves the impression of the solution for the editors.
