---
author: alc
category:
  - sitecore
  - sitecore-8
  - speak
cover:
  alt: sitecore-launchpad-add-application
  image: /wp-content/uploads/2014/12/sitecore-launchpad-add-application.png
date: "2014-12-04T12:37:51+00:00"
guid: http://laubplusco.net/?p=1265
title: Add a SPEAK application to the Sitecore 8 Launchpad
url: /add-speak-application-sitecore-launchpad/

---
_My brain is currently in a complete amoeba state. This is my first blog post since I started my paternity leave some 4 weeks ago. I had been planning to write a post a day but I guess this is not really how taking care of a baby works._

The Launch Pad was introduced in Sitecore 7.1 even though it back then did not contain any applications.

In Sitecore 7.5 the xFile application was added to the launch pad as the only application that shipped with Sitecore.

Sitecore 8 changes this. Now the Launch Pad is the default startup screen for all users.

[![Sitecore 8 Launch Pad](/wp-content/uploads/2014/12/sitecore8launchpad.png)](/wp-content/uploads/2014/12/sitecore8launchpad.png)

This is where you start all applications, from the good-old content editor to the brand new PathAnalyzer application.

_The approach shown in this post is for Sitecore 8 only. I have been using a MVP tech preview build. If this approach changes when Sitecore 8 is released then I will of course update this post accordingly._

## So how do you register your own SPEAK app on the Launch Pad in Sitecore 8?

This is quite simple. The launch pad is actually just a SPEAK application by itself.

It is located in the core database under /sitecore/client/Applications/Launch Pad

When you edit the layout of the Launch Pad item in Sitecore Rocks you will notice the two Launch Pad specific renderings that is called LaunchBar and LaunchTiles

[![launchpadpresentationdetails](/wp-content/uploads/2014/12/launchpadpresentationdetails.png)](/wp-content/uploads/2014/12/launchpadpresentationdetails.png)

The LaunchBar rendering iterates over all child items placed beneath the PageSettings/Buttons item and presumes that these items are LaunchPad-Group items.

It then render out all child items presuming that these are Launchpad-Button items. Each Launchpad-button item has a field for an icon,  a link and a text to represent the button.

Each new Launch-Pad group is placed horizontally in a row while the buttons are rendered out in columns of 3 beneath each group.

Below you see an example of this behavior with 3 new groups that contains various buttons.

[![test-sitecore-launchpad-button-groups](/wp-content/uploads/2014/12/test-sitecore-launchpad-button-groups.png)](/wp-content/uploads/2014/12/test-sitecore-launchpad-button-groups.png)

This setup renders as shown in the screenshot below.

[![sitecore-launchpad-behavior](/wp-content/uploads/2014/12/sitecore-launchpad-behavior.png)](/wp-content/uploads/2014/12/sitecore-launchpad-behavior.png)

Getting back to the actual point of this post then to add a link on the Sitecore Launch Pad for your own SPEAK application then simply do the following.

1. Create a new LaunchPad-Group item beneath /sitecore/client/Applications/Launch Pad/PageSettings/Buttons in the core database ( _optional_)
1. Create a new LaunchPad-Button item beneath the desired group
1. Edit the Text, Link and Icon fields on the Button item. _Notice that it is the field called Icon and not \_Icon that you need to use._

That was it. My next post will be the long awaited one on the Bootstrap SPEAK component library. Stay tuned.
