---
author: alc
category:
  - sitecore
cover:
  alt: 404notfound
  image: /wp-content/uploads/2014/02/404notfound.png
date: "2014-02-10T17:33:32+00:00"
guid: http://laubplusco.net/?p=739
title: Handling 404 in a Sitecore multisite solution and avoid 302 redirects
url: /handling-404-sitecore-avoid-302-redirects/

---
This post shows how to handle a request for a page which is not found, how to avoid having the request being redirected with a 302 status code before a 404 and finally how to use different pages within Sitecore for not found pages in a multisite solution.

The post is based on some code I wrote back in June 2012 and I have seen the concept used several places since then so a lot of you will know it already and might already have it implemented yourselves in some form.

My colleague Brian already wrote a post about the basic concept here in his post [Sitecore 404 without 302](http://briancaos.wordpress.com/2013/03/21/sitecore-404-without-302/) but I will go a bit further into details and extend the code which he is showing.

_The code presented in this post can be downloaded as a module on [Sitecore marketplace](http://marketplace.sitecore.net/Modules/NotFoundItem.aspx). Note that this code is well tested and have been running on several production sites for the last couple of years._

## What is wrong with a 302 status code?

Basically it all comes down to SEO. If a search bot follows a link and receive a 302 then all is good on that url since it has just been temporarily moved to another address.

This means that the requested url's which responds with a 302 is kept in the search indexes which is all expected behaviour even though the real intent was to get the url removed from the index since it no longer exists.

[![Very bad 404 implementation](/wp-content/uploads/2014/02/VeryBad404.png)](/wp-content/uploads/2014/02/VeryBad404.png) To get the url removed from the search indexes then it has to respond right away with a 404 when requested and not with a 302 followed by a 404.

Unfortunately this 302 status code followed by a 404 is seen very often on quite a lot of Sitecore installations due to improper configuration or bad coding. It is actually the out-of-the-box behaviour in Sitecore if you do not change it to use server transfers instead of redirects.

## 404 not found page settings in Sitecore

Sitecore have not just one but two settings in which you can point to an URL on the website when a 404 status code is needed.

```xhtml
      <!--  ITEM NOT FOUND HANDLER
            Url of page handling 'Item not found' errors
      -->
      <setting name="ItemNotFoundUrl" value="/sitecore/service/notfound.aspx" />
      <!--  LAYOUT NOT FOUND HANDLER
            Url of page handling 'Layout not found' errors
      -->
      <setting name="LayoutNotFoundUrl" value="/sitecore/service/nolayout.aspx" />
```

Both of these settings are global for the entire solution and cannot be set for individual sites in a multisite solution.

You can furthermore configure Sitecore to use Server.Transfer instead of Response.Redirect which will avoid the 302 status code.

_**This should always be set to true if you do not use this module or something similar.**_

```xhtml
      <!--  USE SERVER-SIDE REDIRECT FOR REQUEST ERRORS
            If true, Sitecore will use Server.Transfer instead of Response.Redirect to redirect request to service pages
            when an error occurs (item not found, access denied etc).
            Default value: false
      -->
      <setting name="RequestErrors.UseServerSideRedirect" value="true" />
```

It still does not solve that sites in a multisite solution share settings thus share the relative url for when a page is not found.

## Coding a multisite friendly 404 handling

 _What is it we want to acheive exactly?_

When an item is not found by the ItemResolver or if the current item does not have a layout then we would like the response to have status code 404 and we want to show a nice looking page which the editors can control for each site in a solution.

First we have to add a notFoundItem property for site definitions in Sitecore.

We add the key for this property in a constants file and then we write a small service class which can read the property and load it as an item either from an ID, a local or a full path.

```c#
  public class Constants
  {
    public const string NotFoundItemPropertyKey = "notFoundItem";
  }
```

```c#
  public class SiteContextNotFoundItemService
  {
    protected static Item GetItemByShortPath(SiteContext siteContext, string shortPath)
    {
      shortPath = shortPath.StartsWith("/") ? shortPath.Substring(1) : shortPath;
      var fullPath = string.Concat(StringUtil.EnsurePostfix('/', siteContext.StartPath), shortPath);
      return siteContext.Database.GetItem(fullPath);
    }

    public static Item GetItemBySiteProperty(SiteContext siteContext, string propertyKey)
    {
      var property = siteContext.Properties[propertyKey];
      if (string.IsNullOrEmpty(property))
        return null;
      if (ID.IsID(property) || property.StartsWith(Sitecore.Constants.ContentPath))
        return siteContext.Database.GetItem(property);
      return GetItemByShortPath(siteContext, property);
    }

    public static bool HasNotFoundItemKey(SiteContext siteContext)
    {
      return !string.IsNullOrEmpty(siteContext.Properties[Constants.NotFoundItemPropertyKey]);
    }
  }
```

Now when this is ready we implement a HttpRequestProcessor and place in the HttpRequestBegin pipeline after the ItemResolver.

```c#
  public class NotFoundItemResolver : HttpRequestProcessor
  {
    public override void Process(HttpRequestArgs args)
    {
      if (IsValidContextItemResolved()
        || !SiteContextNotFoundItemService.HasNotFoundItemKey(Context.Site)
        || args.LocalPath.StartsWith("/sitecore")
        || RequestIsForPhysicalFile(args.Url.FilePath))
        return;

      Context.Item = GetSiteSpecificNotFoundItem();

      if (Context.Item == null)
        return;
      ItemNotFoundStatusRepository.Set(true);
    }

    protected virtual bool IsValidContextItemResolved()
    {
      if (Context.Item == null || !Context.Item.HasContextLanguage())
        return false;
      return !(Context.Item.Visualization.Layout == null
        && string.IsNullOrEmpty(WebUtil.GetQueryString("sc_layout")));
    }

    protected virtual bool RequestIsForPhysicalFile(string filePath)
    {
      return File.Exists(HttpContext.Current.Server.MapPath(filePath));
    }

    protected virtual Item GetSiteSpecificNotFoundItem()
    {
      return SiteContextNotFoundItemService.GetItemBySiteProperty(Context.Site, Constants.NotFoundItemPropertyKey);
    }
  }
```

 _Note that we first check if a valid context item has been found and exit the processor quickly if one has been found. An item is not valid if it does not have a version in the context language or if it does not have a layout set. Checking for context language version uses [the item extension method from this post.](/sitecore-item-languages-item-extension/ "Sitecore item languages")_

If the request is not for a valid item, or for something residing beneath /Sitecore or for a file then we set the context item to a site specific NotFoundItem.

Now if we would set the response status code to 404 at this early stage then Sitecore will take over control of the request so to avoid this we do not set the status code just yet.

Instead we write a small repository which can get and set a boolean value from the current request items.

```c#
  public class ItemNotFoundStatusRepository
  {
    public static bool Get()
    {
      return HttpContext.Current.Items[Constants.NotFoundItemPropertyKey] != null && (bool) HttpContext.Current.Items[Constants.NotFoundItemPropertyKey];
    }

    public static void Set(bool status)
    {
      HttpContext.Current.Items[Constants.NotFoundItemPropertyKey] = status;
    }
  }
```

Then we implement yet another HttpRequestProcessor which we place in the end httpRequestProcessed pipeline.

```c#
  public class SetNotFoundStatusCode : HttpRequestProcessor
  {
    public override void Process(HttpRequestArgs args)
    {
      if (!ItemNotFoundStatusRepository.Get())
        return;
      HttpContext.Current.Response.StatusCode = (int) HttpStatusCode.NotFound;
      HttpContext.Current.Response.TrySkipIisCustomErrors = true;
    }
  }
```

Finally we add an include config file for patching in the processors in the pipelines.

```xhtml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <pipelines>
      <httpRequestBegin>
        <processor
          patch:before="processor[@type='Sitecore.Pipelines.HttpRequest.LayoutResolver, Sitecore.Kernel']"
          type="LaubPlusCo.NotFoundItem.InfraStructure.NotFoundItemResolver, LaubPlusCo.NotFoundItem" />
      </httpRequestBegin>
      <httpRequestProcessed>
        <processor type="LaubPlusCo.NotFoundItem.InfraStructure.SetNotFoundStatusCode, LaubPlusCo.NotFoundItem" />
      </httpRequestProcessed>
    </pipelines>
  </sitecore>
</configuration>
```

To get it working we of course need to fill in the notFoundItem Property on the sites on which the module should be used.

_Example_

```xhtml
<site name="website" notFoundItem="/404" ...
```

## Sum up

A nice feature of this module is that you will get a 404 and a nice looking page whatever nonsense you type into the address bar if it does not resolve to a valid item or file. Just as when you use the server side error redirects.

[![Good 404 on a nice looking item](/wp-content/uploads/2014/02/Good404.png)](/wp-content/uploads/2014/02/Good404.png)

As mentioned in the beginning this code is somewhat old but I think it is extremely useful and it solves the somewhat lacking normal 404 error handling in an elegant manner.

The module can be downloaded from Sitecore marketplace here [NotFoundItem](http://marketplace.sitecore.net/Modules/NotFoundItem.aspx)

Returning a 404 when no layout exists on the context item like I showed here breaks an out-of-the-box feature in Sitecore which I never have seen anyone use. That is the _DefaultLayoutFile_ setting which can point to a fallback file to use as layout if no presentation details has been set on an item. Has anyone ever used this setting?
