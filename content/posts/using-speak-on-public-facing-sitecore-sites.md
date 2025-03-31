---
author: alc
category:
  - sitecore
date: "2014-05-08T08:27:04+00:00"
draft: "true"
guid: http://laubplusco.net/?p=898
title: Using SPEAK on public facing Sitecore sites
url: /

---
_This code is just made for fun and not for production sites._

SPEAK applications reside in the core database and the framework is for now intended for creating applications within Sitecore and not for creating public facing websites.

But what if you would like to use SPEAK on a public facing website build using Webforms today?

Wait no longer, it is possible to do just by writing a bit of wrapping.

The approach is a bit dubious but it does work.

## Render MVC View Renderings in a Webform contexts

First challenge is that most of our public facing  websites are written using Webforms so we need to wrap MVC View renderings in a Webform context.

To do this I created the following Webcontrol which wraps a MVC View Rendering in a Sitecore WebControl.

This is highly inspired by / copied from this post by Hedgehog, thanks for sharing this.

## SPEAKifying a page

To SPEAKify a page, all you would need to do in the ideal world would be to include the PageCode view rendering.

The current PageCode.cshtml file contain some references to the SPEAK-layout item template which makes it fail hard if a field called "BrowserTitle" is not present on the current item. Furthermore it calls some HtmlHelper extension methods which also makes some presumptions about the context which makes this rendering fail when it is not on a SPEAK-layout item.

It seems like the PageCode file has got too much responsibility, I can't see what the BrowserTitle has to do with the PageCode.

Please, Sitecore SPEAK team, fix this so the PageCode only renders out the PageCode. Do not mix responsibilities.

To fix this for now, simply copy the PageCode view rendering and the PageCode.cshtml file. Then remove the failing lines from the new PageCode.

And now it is ready to be used on public facing sites.

## Including the PageCode view rendering from Core.

Next thing is that we need to include a view rendering from core on an item which is not in core.

To do this I made a WebControl which inherits from the MvcWebControl and renders out the copied PageCode view rendering.

Then I added the WebControl in the master database so we now can include the PageCode on any page.

## Rendering SPEAK controls

This is simply a bit of rinse, repeat of how it was done with the PageCode.
