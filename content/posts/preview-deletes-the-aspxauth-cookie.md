---
author: alc
category:
  - errors
  - sitecore
  - sitecore-6.6
  - tips
date: "2013-10-23T14:50:29+00:00"
guid: http://laubplusco.net/?p=202
title: Preview deletes the .aspxauth cookie
url: /preview-deletes-aspxauth-cookie/

---
I've been working a bit on an extremely weird support case the last couple of days.

When the editors on a site clicked on preview from either the content editor or the page editor then the page ended up in a redirect loop.

[![redirectloop](/wp-content/uploads/2013/10/redirectloop.png)](/wp-content/uploads/2013/10/redirectloop.png)

### Who stole my cookie?

Using Fiddler I quickly noticed that the .ASPXAUTH cookie was removed from the session as soon as preview was clicked and what actually happened was the user being redirected infinitely to the login page until the browser crashed (chrome crashes nice, IE just fails miserably).

[![missingcookie_1](/wp-content/uploads/2013/10/missingcookie_1-1024x840.png)](/wp-content/uploads/2013/10/missingcookie_1.png)

Then I tried to reproduce the issue in our local dev environment and everything worked fine. I then tried to attach to the production databases from my own environment and then the error occurred.

I debugged all our code which played around with cookies, set a breakpoint everywhere we called logout etc. etc. no results at all.

I started suspecting some editor of having played around with some security setting which caused the issue and went through each and every possible security setting I could think of. Still no result.

As a last resort I tried creating a support case at Sitecore support explaining the issue.

Before I got a response back from support I thought what about locked items or something weird like that?

In the content editor I right-clicked in the gutter and selected to show locked items, publishing warnings and so on.

[![publishingwarnings](/wp-content/uploads/2013/10/publishingwarnings.png)](/wp-content/uploads/2013/10/publishingwarnings.png)

Then I noticed a publishing warning on the Sitecore root item.

I clicked on publishing restrictions and here goes. Some one had made the Sitecore root item un-publishable.

Checking the checkbox back on again and resetting the app pool made everything back to normal again.

[![publishableunticked](/wp-content/uploads/2013/10/publishableunticked.png)](/wp-content/uploads/2013/10/publishableunticked.png)

I am not sure why someone would steal my cookie just because the root item is unpublishable, that is really an unexpected symptom.

Try it yourself on a random Sitecore solution, simply uncheck the Publishable checkbox on the Sitecore root and try to preview a page. Look in Fiddler or just the Resource pane in dev tools and see how the cookie just magically disappears.
