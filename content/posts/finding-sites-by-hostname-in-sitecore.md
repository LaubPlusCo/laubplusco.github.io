---
author: alc
category:
  - sitecore
date: "2014-10-24T17:23:28+00:00"
draft: "true"
guid: http://laubplusco.net/?p=896
title: Finding sites by hostname in Sitecore
url: /

---
The current SiteContext, Sitecore.Context.Site is resolved by looking up the most qualified site using the current request hostname.

This poses quite a lot of issues when using the page editor if your renderings are dependent on the site context in some way.

To solve this we have an extension method on an item that returns the most qualified site using the site path and the site root path.

\[CODE\]

This works most of the time but not always. This approach requires all sites in the solution to reside independently.

A colleague of mine had an issue with our general approach extension method today where this approach was not sufficient.

In the, actually quite common, case where several site definitions point to the same site structure we also have to check the hostname of the current request.

Then I remembered the WildCardParser from the Sitecore API.

## WildCardParser to the rescue

The WildCardParser is a static "helper" class that contain a boolean method which can compare two hostnames.

It is not very well known since its main objective has just been to compare the current request hostname with the hostName attribute from the Sitecore site definitions thus lookup the current Sitecontext (see the SiteResolver processor in the httpRequestBegin pipeline).

To solve our "resolve sitecontext by item" issue I added the following check.

\[CODE\]

In short, first we find all site contexts which this item reside in using the item path. Then we check if this is just one site or none, if so we return this or null. If the item can belong to more than one site then we check if we can use the current requests hostname to find the most qualified site of the most qualified sites.

This approach is not a 100 % solid, it still requires a somewhat stringent content architecture. It will work in most cases though, which is why this is quite a solid extension method to keep in your repository.

_I just remembered that I had this post lying in my drafts until I read [this great short post by my colleague Uli](http://reasoncodeexample.com/2014/10/24/resolving-the-sitecontext-by-url)._
