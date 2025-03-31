---
author: alc
category:
  - sitecore
  - tips
cover:
  alt: updateinstallationwizard
  image: /wp-content/uploads/2014/09/updateinstallationwizard.png
date: "2014-09-30T21:12:38+00:00"
guid: http://laubplusco.net/?p=1074
title: A quick tip on upgrading Sitecore fast and easy
url: /quick-tip-upgrading-sitecore-fast-easy/

---
Today my colleague [@alinulms](https://twitter.com/alinulms) and I had the "fun" task of updating an old Sitecore 6.5 to version 7.2.

_Actually we really want to upgrade the solution to Sitecore 7.5 but we are still waiting for Sitecore. I really hope 7.5 will be officially released any time real soon._

I've upgraded Sitecore installations at least a trillion times before (or so it feels) and depending on the version, build environment etc. I've used a number of different approaches. The approach we used today was extremely effective so I thought that it deserves a blog post.

**_The tip_** is simply, don't upgrade the actual solution, instead point a clean Sitecore on the solution databases, upgrade this solution, then fix your code and configuration afterwards.

## Back when migrating content to a new installation was the way to go

When I've upgraded from Sitecore 5.3 - 6.3 to a version 6.4 or higher I would almost always simply move the content to a new version and then upgrade my codebase.

I've simply setup a new Sitecore with databases and everything. Then I moved all the production content into the clean new Sitecore using either Sitecore packages or later serialization.

When the content was in the new Sitecore then I've merged the code and configuration in as well. How this can be done depends on the solution setup. Here at Pentia we simply use our buildlibrary and subversion to do this for us which makes this task a real no-brainer so we can focus on getting the code updated to fit the upgraded API.

Last step was then to get the new databases out into production again and maybe sync any minor content changes if the upgrade had been slow.

The reason for not following the normal Sitecore upgrade path found on sdn is the amount of packages, sql scripts, configuration changes and other steps along the way. There is a lot of pitfalls when following the steps one by one.

Moving content does not work well on large solutions, it becomes too time consuming compared with installing the upgrade packages.

Some great tools such as [TDS](https://www.hhogdev.com/Downloads/Team-Development-for-Sitecore.aspx) and [Razl](https://www.hhogdev.com/products/razl.aspx) from Hedgehog or [Unicorn](https://github.com/kamsar/Unicorn) can help restoring the content into clean databases. These tools are not always available to you as a developer and again the amount of especially media data can become an obstacle.

Migrating content to a new solution can be a bit reckless since the moved items are not updated and new versions of the UpdateInstallationWizard performs some post installation steps which fixes some field values on standard fields.

Luckily Sitecore made the upgrades a tiny bit easier in the later versions, a bit more stable. Now it is simply installing the initial releases one by one and then finally the latest version.

## Upgrading an out-of-the-box Sitecore webroot with a database full of content

This was the approach we went for today and wow it was actually fast.

The solution we were upgrading is relative big, the master database is about 20 GB and we do not use neither TDS nor Unicorn on this solution. Moving the content via packages is not an option due to the huge amount of data. Instead we followed the upgrade path found on SDN but with a small twist that made it go quick and easy since we could ignore most of the instructions.

Here is a quick recipe based on what we did today.

1. **Copy databases** \- Make a copy of all the solution Sitecore databases preferably using detach/attach. Include webforms databases etc.
1. **Install a temporary clean Sitecore -** Take a clean Sitecore webroot, same version as the solution, point the connection strings to the copied databases and setup the site in IIS.
1. **Install upgrade packages** \- Download the required packages from sdn. Log into Sitecore and go to the <new site>/sitecore/admin/UpdateInstallationWizard.aspx page. Install the .upgrade packages from SDN one-by-one and run SQL scripts as needed. Also update config if needed. This can be done by simply copying in the new config files.
1. **Drink a lot of coffee -** while waiting.. Each upgrade package takes a few minutes to install
1. **Install module upgrades** \- When the solution is upgraded to the desired Sitecore version then install all module  updates if any (webforms, ecm) again using the UpdateInstallationWizard.
1. **Delete the temporary site** \- Delete the temporary webroot
1. **Merge your code in** \- Get a clean webroot of the new Sitecore version and move your code into this. This step depends on how you are working with Sitecore and cannot really be described. Just get all your code and markup back where it belongs. Rebuild indexes and link database.
1. **Fix code errors**\- some tips

    1. The major API change between 6 and 7 is of course Sitecore.Search. This can be a big task depending on the solution.
    1. Remember to remove any old Sitecore support patches etc. which is no longer valid.
    1. All Visual Studio projects which are dependent on the Sitecore kernel need to be updated to target framework 4.5 or later when upgrading from 6 to 7.x
    1. Remember the [Sitecore AntiCsrf module and that Text fields are suddenly output-escaped](/sitecore-update-bummer/). This can sometime cause trouble when upgrading.
1. **Rinse Repeat** \- Repeat step 1 - 6 in other environments (preprod, production)
1. **Deploy** \- Deploy the upgraded website in other environments

Step 9 and 10 is of course dependent on the specific customer setup. If possible then make a copy of the production databases , tell the editors to stop working for some hours, run the upgrades, drink a lot of coffee, deploy all the new code to a new folder on the different servers. Finally make a clean switch by simply changing the website root path in IIS on all servers to the new folder. No downtime at all, just a site restart. This approach requires that you can freeze the content while upgrading though so it need to be done fast.

Using this approach you will never really leave the UpdateInstallationWizard.aspx so you don't always have to mind all the other steps that Sitecore instruct you to do such as  deleting temporary files etc. a lot of these steps can be skipped. Updating config can be done by copying in the new config files instead of manually merging in the changes.

**DISCLAIMER**: _ALWAYS READ ALL UPGRADE NOTES FROM SITECORE AND CONSIDER IF THEY HAVE TO BE PERFORMED ON YOUR SOLUTION._

The main point is simply to upgrade an empty Sitecore so you do not have to fix your code each and every step on the way to the desired version.

If you have any experience in upgrading Sitecore, good or bad please share in the comments below.

## A sneak-peak on Sitecore 8

I just checked my Sitecore 8 Tech preview and the UpdateInstallationWizard has not been updated.. Or at least it looks the same. Perhaps Sitecore will do some magic before the final release, I sure hope that they at least will get rid of the very long checklists on sdn. These manual tasks should almost all be automated.
