---
author: ap
category:
  - sitecore
date: "2014-11-11T10:18:38+00:00"
guid: http://laubplusco.net/?p=1252
title: Sitecore 7.5 Analytics templates bug
url: /bug-hunt-sitecore-7-5/

---
Found an interesting bug today in a Sitecore 7.5 installation and I think it is worth sharing. I have to mention that Sitecore Support has been notified and they are working on an official fix.

I have noticed this issue after I upgraded from version 7.2 but also on clean installations. If you navigate to this path in Sitecore: _/sitecore/system/Marketing Center/Engagement Plans_ you will see that all the display names of the engagement plans, states, conditions and actions are _"\_\_Standard Values"._ [![analyticsTemplateBug](/wp-content/uploads/2014/11/analyticsTemplateBug.png)](/wp-content/uploads/2014/11/analyticsTemplateBug.png)

This is because the templates of all these items have the Display name field set on their \_\_Standard Values to be "\_\_Standard values" or their translations (f.ex. \_\_Richtwerte in German language).

[![standardValuesDisplayName](/wp-content/uploads/2014/11/standardValuesDisplayName-300x91.png)](/wp-content/uploads/2014/11/standardValuesDisplayName.png)

Those fields should be empty off course.

I have to give credits to my colleague [Brian](http://briancaos.wordpress.com) as he was by my side when we discovered this bug.
