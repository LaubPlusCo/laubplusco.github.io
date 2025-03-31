---
author: alc
category:
  - sitecore
  - sunday
  - tips
cover:
  alt: custom_caches_in_Sitecore
  image: /wp-content/uploads/2014/10/custom_caches_in_Sitecore.png
date: "2014-10-26T13:35:12+00:00"
guid: http://laubplusco.net/?p=1202
title: How to create a custom cache in Sitecore
url: /create-custom-cache-sitecore/

---
When coding a Sitecore solution you sometimes need a cache to store some values.

I've seen a lot of different cache implementations in Sitecore solutions that do not use the Sitecore API. Such an implementation could for example use a static or perhaps a singleton some even go further and use a custom database or a nosql document database such as RavenDB or Mongo.

It is very rare that these implementations could not simply be replaced by using the built-in Sitecore caching API.

To implement a simple cache you simply derive from the Sitecore class called CustomCache.

```c#
public class DesignBundleCache : CustomCache
{
  public DesignBundleCache(long maxSize)
    : base("DesignResources.DesignBundleCache", maxSize)
  {
  }
}
```

You need to create a class constructor that calls the base constructor with the name of the cache and the max allowed size.

Then you can either wrap access to your cache in a static like below

```c#
public static readonly DesignBundleCache DesignBundleCache
  = new DesignBundleCache(StringUtil.ParseSizeString(Settings.GetSetting("DesignResources.DesignBundleCacheMaxSize", "50MB")));
```

 _Notice the call to Sitecore.StringUtil.ParseSizeString( .. ), this method converts a string such as 512KB or 10MB to a long representing the size in bytes. This makes configuration of sizes easy to read._

Or you can simply instantiate the cache each time you need it. This will make Sitecore look for a cache with that name and then take the existing cache instance if any and wrap it as the inner cache of your custom cache. Calling the constructor each time you need to cache can be fine if your cache constructor do not have parameters, otherwise the static approach is better.

The most basic usage of a custom cache is to use it for storing string values. Then you just call GetString and SetString on the CustomCache base class from Get and Set methods within your derived class.

```c#
  public string Get(string cacheKey)
  {
    return GetString(cacheKey);
  }

  public void Set(string cacheKey, string value)
  {
    SetString(cacheKey, value);
  }
```

If you need to cache complex objects you can do this as well. Then you have to write a class that implements the ICacheable interface.

```c#
public class DesignResourceFile : ICacheable
{
  public byte[] Content { get; set; }
  public DateTime LastModified { get; set; }
  public string ETag { get; set; }
  public string MimeType { get; set; }

  public long GetDataLength()
  {
    return Content.Length;
  }

  public bool Cacheable { get; set; }

  public bool Immutable
  {
    get { return true; }
  }

  public event DataLengthChangedDelegate DataLengthChanged;
}
```

Then you call the Get and Set methods of the CustomCache base class from within your derived class.

```c#
public DesignResourceFile Get(string cacheKey)
{
  return (DesignResourceFile) GetObject(cacheKey);
}

public void Set(string cacheKey, DesignResourceFile designResourceFile)
{
  SetObject(cacheKey, designResourceFile);
}
```

A CustomCache will be maintained by Sitecore when it is instantiated. This is done by the CustomCache base constructor that registers the instantiated cache class. As mentioned previously only one cache can exists with the same name. If another class is instantiated with the same name but different max size then it will still be the original cache instance that is used and the max size will not be changed.

Now when Sitecore is maintaining your cache you are also able to see how it is being utilized on the admin page /sitecore/admin/cache.aspx

[![customcachesinsitecore](/wp-content/uploads/2014/10/customcachesinsitecore.png)](/wp-content/uploads/2014/10/customcachesinsitecore.png)

The cache is cleared when CacheManager.ClearAllCaches(); is called and can also be cleared by itself by calling Clear();

It is really super easy to create your own custom caches in Sitecore. Of course beware of the memory consumption and do please configure your cache sizes to match with the environments that the solution runs in.

Sitecore caching does not distribute across different Sitecore instances. Not yet at least, they are right now in memory per instance. If this changes one day in the future then I am sure that Sitecore will do their best to keep the development interface to the caches similar so if they choose to change their underlying technologý that our custom caches still will work with little or no modification.

This is the last post I will write today, see the two previous ones as well, the first about [matching hostname wildcards in Sitecore](/matching-wildcards-sitecore/) and the other about [sending mails using the Sitecore API](/sending-emails-sitecore/).
