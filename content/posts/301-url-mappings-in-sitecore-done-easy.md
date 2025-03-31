---
author: alc
category:
  - architecture
  - link-database
  - modules
  - multisites
  - pipelines
  - sitecore
date: "2013-10-14T11:06:08+00:00"
guid: http://laubplusco.net/?p=135
title: 301 URL mappings in Sitecore done easy
url: /url-mappings-in-sitecore-done-easy/

---
When migrating an existing website to a new Sitecore solution the customer typically want to retain their Google Page rank earned by links pointing to their site.

To transfer the page ranks of old pages to new ones the response needs a 301 permanent redirect status code and then redirect to a new page which content some what corresponds to the content found on the old url.

_**A simple example**_

www.somesite.org/about-us.html was an url on the old site. On the new site the about us page is located on www.somesite.org/en/about/who-we-are

Now let say that Google Webmaster tools tells us that 50 links exist on the web to the old url, then the next time these links are crawled it is important that a 404 is not returned. Instead a 301 will redirect to the new page.

## URL mappings in Sitecore

Okay, what we want to achieve is that if an item is not found on an url by the ItemResolver in the httpRequestBegin pipeline then we want to check if it is because it is an url from the old website which is being requested.

To do this we first make a pipeline processor for the httpRequestBegin pipeline which we patch right after the ItemResolver or maybe patch before the LayoutResolver just in case some other processor needs to run after the ItemResolver.

```xhtml
<processor type="[NAMESPACE].UrlRedirectResolver, [ASSEMBLY]" />
```

In this processor we will first check that the request is not made for sitecore if so then exit fast.

```c#
public override void Process(HttpRequestArgs args)
{
  if (args.Context.Request.Url.OriginalString.ToLower().Contains("/sitecore"))
    return;
  if (Context.Item == null)
    CheckUrlMappingAndRedirect();
}
```

Otherwise we first look up a url mapping folder in Sitecore. If it is a multisite solution then we'll look up the url mapping folder in the site context using the extension method from [my previous post](/item-extensions-get-referrers-as-items/ "Get referrers as items through the link database").

```c#
private static void CheckUrlMappingAndRedirect()
{
  var urlMappingFolderTemplate = DatabaseService.ActiveDatabase.GetItem(Constants.Templates.UrlMappingsRootFolder);
  if (urlMappingFolderTemplate == null)
    return;

  var urlMappingFolder = urlMappingFolderTemplate.GetReferencesInContextSite().FirstOrDefault();

  if (urlMappingFolder == null)
    return;

  while (urlMappingFolder.Parent.TemplateID == new ID(Constants.Templates.UrlFolder))
    urlMappingFolder = urlMappingFolder.Parent;

  Item urlMappingItem = null;
  var path = HttpContext.Current.Request.Url.LocalPath;
  while (urlMappingItem == null)
  {
    urlMappingItem = GetItemFromPath(path, urlMappingFolder.Paths.FullPath);
    if (string.IsNullOrEmpty(path))
      break;
    if (urlMappingItem == null)
      path = path.Substring(0, path.LastIndexOf('/'));
  }

  if (urlMappingItem == null)
    return;

  if (!path.Equals(HttpContext.Current.Request.Url.LocalPath) && !urlMappingItem.GetCheckBoxValue(new ID(Constants.Fields.PageRedirect.IsWildCardPageRedirect)))
    return;

  var itemToRedirect = urlMappingItem.GetDropLinkSelectedItem(new ID(Constants.Fields.PageRedirect.ItemToRedirect));
  if (itemToRedirect == null)
    return;
  HttpContext.Current.Response.RedirectPermanent(LinkManager.GetItemUrl(itemToRedirect), true);
}

private static Item GetItemFromPath(string path, string rootPath)
{
  var itemRelativePath = StringUtil.RemovePostfix('/', path);
  return DatabaseService.ActiveDatabase.GetItem(string.Concat(rootPath, itemRelativePath));
}
```

When the url mapping folder has been resolved we try to see if an item exist beneath this folder on the same relative path as the one being requested. If not we remove the last part of the relative path and try again until an item is found or breaks out of the pipeline if nothing is matched.

If an item is found somewhere along the hierarchy we check if it can be used as a wildcard redirect. If so we redirect to that one otherwise we just continue so Sitecore handles a 404.

The benefits of this code is that we "fake" the old website structure using the relative paths of items residing beneath the url mapping folder for that specific web site context. An alternative to this would be creating a custom database and interface for the editors where this approach seems much more powerful.

In Sitecore we create a template structure as follows:

[![url-mapping-templates](/wp-content/uploads/2013/10/url-mapping-templates.png)](/wp-content/uploads/2013/10/url-mapping-templates.png)

Where the \_\_PageRedirect base template contain the following two fields:

[![page-redirect-item](/wp-content/uploads/2013/10/page-redirect-item.png)](/wp-content/uploads/2013/10/page-redirect-item.png)

And create the url mappings folder for the website:

[![url-mappings-folder-in-sitecore](/wp-content/uploads/2013/10/url-mappings-folder-in-sitecore.png)](/wp-content/uploads/2013/10/url-mappings-folder-in-sitecore.png)

Creating all the url's can be a very tedious task. If the website which is being migrated is large it can pay off to use a Google GSA or similar to index the old site, export all url's to csv and then write a small script which runs through the csv and creates the corresponding page redirect items.

The wildcard option makes it easy for editors to redirect whole sections of their old website to a new page. Often it is not possible to make a 1 - 1 mapping between a new site and an old one.

That is it, plain and simple. If you want the source code in a project along with the Sitecore templates write a comment on this post and I'll prepare it.
