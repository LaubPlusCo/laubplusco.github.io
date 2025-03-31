---
author: alc
category:
  - modules
  - multisites
  - pipelines
  - sitecore
date: "2014-02-03T11:00:49+00:00"
draft: "true"
guid: http://laubplusco.net/?p=540
title: Overriding the SiteProvider in Sitecore
url: /

---
About a year and a half ago I decided to make my own small lightweight multi-site module for Sitecore. Not that there is anything wrong with the [multisite manager](http://marketplace.sitecore.net/en/Modules/Multiple_Sites_Manager.aspx) I just found it to somewhat over-complicate some things which otherwise should be relatively simple.

What I wanted to achieve was simply to be able to add new sites by creating new site items in Sitecore with no need for editing the web.config file.

I've now updated and tested the implementation on Sitecore 7.1 and made the code into a module. This is something I have been planning for a long time, I actually promised it would be available asap back in October to a meeting with the [Danish Sitecore Developer Group](http://www.meetup.com/Danish-Sitecore-Developer-Group)

## Overriding the SiteProvider

Sitecore ships with an implementation of the abstract class SiteProvider which is called ConfigSiteProvider. This class has only changed very little over time and is the foundation for looking up sites in Sitecore.

We will of course still keep the ConfigSiteProvider as base class for our new class otherwise all the Sitecore system sites would stop working.

We call the new class ItemSiteProvider since it reads sites from items.

```c#
  public class ItemSiteProvider : ConfigSiteProvider
  {
    public override Site GetSite(string siteName)
    {
      var site = SiteRepository.GetSiteByName(siteName) ?? base.GetSite(siteName);
      return site;
    }

    public override SiteCollection GetSites()
    {
      var sites = SiteRepository.GetAllSites();
      sites.AddRange(base.GetSites());
      return sites;
    }
  }
```

We keep reading of sites from items in a repository which takes precedence over the sites from config which is retrieved by calling the base methods if nothing is returned from the repository.

```c#
  public class SiteRepository
  {
    public static SiteCollection GetAllSites()
    {
      var siteCollection = new SiteCollection();
      var websiteItems = WebSiteItemRepository.Get();
      if (websiteItems == null || !websiteItems.Any())
        return siteCollection;
      foreach (var websiteItem in websiteItems.Where(s => s != null))
      {
        siteCollection.Add(GetFromWebsiteItem(websiteItem));
      }
      return siteCollection;
    }

    public static Site GetSiteByName(string siteName)
    {
      var websiteItems = WebSiteItemRepository.Get();
      if (websiteItems == null || !websiteItems.Any())
        return null;
      var websiteItem = websiteItems.FirstOrDefault(item => item.Name.Equals(siteName, StringComparison.InvariantCultureIgnoreCase));
      return websiteItem == null ? null : GetFromWebsiteItem(websiteItem);
    }

    public static Site GetFromWebsiteItem(Item websiteItem)
    {
      if (websiteItem == null)
        return null;
      var websiteProperties = WebSiteItemRepository.GetWebSiteItemProperties(websiteItem);
      return websiteProperties == null ? null : new Site(websiteItem.Name, websiteProperties);
    }
  }
```

The SiteRepository calls a WebSiteItemRepository to retrieve items which inherits from a specific website base template. The repository is also used to read properties from these items.

```c#
  public class WebSiteItemRepository
  {
    public static IList<Item> Get()
    {
      var websiteBaseTemplate = SiteConfigurationService.GetMultiSiteDatabase().GetItem(new ID(Constants.Templates.WebSiteBaseTemplate));
      if (websiteBaseTemplate == null)
        return new Item[0];
      var websiteTemplates = websiteBaseTemplate.GetReferrersAsItems();
      if (websiteTemplates == null || !websiteTemplates.Any())
        return new Item[0];
      if (websiteTemplates.Length == 0) return null;
      var websiteItems = new List<Item>();
      foreach (var websiteTemplate in websiteTemplates.Select(template => template.GetReferrersAsItems().Where(i => i.Paths.FullPath.StartsWith(Sitecore.Constants.ContentPath, StringComparison.InvariantCultureIgnoreCase))))
      {
        websiteItems.AddRange(websiteTemplate);
      }
      return websiteItems.Where(i => i.Name != "__Standard Values" && i.IsDerived(new ID(Constants.Templates.WebSiteBaseTemplate))).Distinct().ToList();
    }

    public static Item GetByPath(string path)
    {
      var websiteItems = Get();
      if (websiteItems == null || !websiteItems.Any()) return null;
      return websiteItems.FirstOrDefault(rootItem => path.StartsWith(rootItem.Paths.FullPath, StringComparison.InvariantCultureIgnoreCase));
    }

    public static Item GetByHostName(string hostName)
    {
      var websiteItems = Get();
      if (websiteItems == null || !websiteItems.Any()) return null;
      return websiteItems.FirstOrDefault(item => MatchItemHostName(hostName, item[new ID(Constants.Fields.HostName)]));
    }

    public static StringDictionary GetWebSiteItemProperties(Item websiteItem)
    {
      var siteSettings = SiteConfigurationService.GetMultiSiteSettings();
      var itemSettings = GetSiteSettingsFromItem(websiteItem);
      if (itemSettings == null)
        return null;
      siteSettings.AddRange(itemSettings);
      return siteSettings;
    }

    private static StringDictionary GetSiteSettingsFromItem(Item item)
    {
      Assert.IsNotNullOrEmpty(item.Name, "Cannot get website settings from an item with no name");
      var startItem = (new InternalLinkField(item.Fields[new ID(Constants.Fields.StartItemAndRootPath)])).TargetItem;
      if (startItem == null)
        return null;
      var paths = startItem.Paths.FullPath.Split('/');
      var languageItem = (new InternalLinkField(item.Fields[new ID(Constants.Fields.Language)])).TargetItem;
      var siteConfiguration = new StringDictionary
                                {
                                  {Constants.SiteProperties.Name, item.Name},
                                  {Constants.SiteProperties.Language, languageItem.Name},
                                  {Constants.SiteProperties.HostName, item[new ID(Constants.Fields.HostName)]},
                                  {Constants.SiteProperties.TargetHostName, item[new ID(Constants.Fields.TargetHostName)]},
                                  {Constants.SiteProperties.RootPath, StringUtil.EnsurePrefix('/', string.Join("/", paths.Take(paths.Length - 1).ToArray()))},
                                  {Constants.SiteProperties.StartItem, StringUtil.EnsurePrefix('/', paths[paths.Length - 1])},
                                  {"database", "web"}
                                };
      var customProperties = item[new ID(Constants.Fields.CustomProperties)];
      if (!string.IsNullOrEmpty(customProperties))
        AddCustomProperties(customProperties, siteConfiguration);
      return siteConfiguration;
    }

    private static void AddCustomProperties(string customProperties, StringDictionary siteConfiguration)
    {
      var properties = customProperties.Split('|').Select(property => property.Split('=')).Where(keyValue => keyValue.Length == 2).ToArray();
      if (!properties.Any())
        return;
      foreach (var keyValue in properties)
      {
        siteConfiguration.Add(keyValue[0], keyValue[1]);
      }
    }

    private static bool MatchItemHostName(string hostName, string hostnameToMatch)
    {
      if (hostName.Length == 0 || hostnameToMatch.Length == 0)
        return false;
      hostName = hostName.ToLowerInvariant();
      return WildCardParser.Matches(hostName, WildCardParser.GetParts(hostnameToMatch));
    }
  }
```

 _Note that I use the ItemExtension method [GetReferrersAsItems](/item-extensions-get-referrers-as-items/ "Get referrers as items through the link database") to retrieve all items which is based on a template which is based on the WebSiteBaseTemplate through the link database. Alternately Lucene could be used but the link database will in this case be much faster as long as there is not hundred or thousands of sites. I also use the [IsDerived](/does-a-sitecore-item-derive-from-a-template/ "Does a Sitecore item derive from a template") extension method to check if the item is based on the template._

To limit the amount of settings / properties which the editors need to set on website items I have made a new multi site configuration node under /configuration/sitecore which looks like this.

```xhtml
      <multiSites>
        <mode>on</mode>
        <language>en</language>
        <database>web</database>
        <content>master</content>
        <contentLanguage>en</contentLanguage>
        <masterDatabase>web</masterDatabase >
        <virtualFolder>/</virtualFolder>
        <physicalFolder>/</physicalFolder >
        <filterItems>false</filterItems>
        <registryCacheSize>0</registryCacheSize>
        <viewStateCacheSize>0</viewStateCacheSize>
        <xslCacheSize>5MB</xslCacheSize>
        <filteredItemsCacheSize>2MB</filteredItemsCacheSize>
        <loginPage></loginPage >
        <requireLogin>false</requireLogin>
        <cacheHtml>true</cacheHtml>
        <domain>extranet</domain >
        <port>80</port>
        <enableDebugger>true</enableDebugger >
        <enablePreview>true</enablePreview >
        <enableWorkflow>true</enableWorkflow >
        <enableAnalytics>true</enableAnalytics >
        <allowDebug>true</allowDebug >
        <browserTitle>Multisite platform</browserTitle >
        <disableClientData>false</disableClientData >
        <disableXmlControls>false</disableXmlControls>
      </multiSites>
```

These settings are read by a SiteConfigurationService from config.

```c#
  internal class SiteConfigurationService
  {
    internal static StringDictionary GetMultiSiteSettings()
    {
      var siteConfiguration = new StringDictionary();
      foreach (var multiSitePropertyNode in Factory.GetConfigNodes("multiSites/*").Cast<XmlNode>())
      {
        if (multiSitePropertyNode != null && multiSitePropertyNode.FirstChild != null && multiSitePropertyNode.FirstChild.NodeType == XmlNodeType.Text)
          siteConfiguration.Add(multiSitePropertyNode.Name, multiSitePropertyNode.FirstChild.Value);
      }
      return siteConfiguration;
    }

    internal static Database GetMultiSiteDatabase()
    {
      var databaseSetting = Factory.GetConfigNodes("multiSites/*").Cast<XmlNode>().FirstOrDefault(d => d.Name == "database");
      return Factory.GetDatabase(databaseSetting == null ? "web" : databaseSetting.FirstChild.Value);
    }
  }
```

## Resolving sites

Now when the website items can be read the next thing is to resolve the correct site on each and every request. To do this we override the SiteResolver.

CODE CODE CODE

## Clearing the cache

Finally we need to make sure that the cache whenever a publish is made. This is done by overriding the HtmlCacheClearer

```c#
  public class MultiSiteHtmlCacheClearer : HtmlCacheClearer
  {
    public new void ClearCache(object sender, EventArgs args)
    {
      Sites.AddRange(WebSiteItemRepository.Get().Select(s => s.Name).ToArray());
      base.ClearCache(sender, args);
    }
  }
```

Which is patched into config instead of the HtmlCacheClearer like this:

CONFIG CONFIG CONFIG

Setting up the template

In Sitecore ....

That was it. Not completely simple but still quite an easy implementation. The code is running in production today and works fine.

Now when I made the code into a module I would really like to get rid of all the static methods to make the individual classes override-able and also do some refactoring to make class responsibilities even clearer.

The module does not handle setting up the new websites in IIS so this still has to be done manually.

When I finished the last touches the code will be made available on gitHub and as a ready module on Sitecore market place.

Follow me on [twitter](https://twitter.com/AndersLaub "Follow me on Twitter") to be notified when it is ready for download.
