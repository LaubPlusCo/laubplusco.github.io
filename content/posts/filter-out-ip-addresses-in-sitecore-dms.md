---
author: alc
category:
  - analytics
  - sitecore-dms
  - tips
cover:
  alt: oie_121156406SOzBwIF
  image: /wp-content/uploads/2014/03/oie_121156406SOzBwIF.jpg
date: "2014-03-12T10:56:19+00:00"
guid: http://laubplusco.net/?p=821
title: Filter out IP addresses in Sitecore DMS
url: /filter-ip-addresses-sitecore-dms/

---
Today I was asked by a customer to filter out their internal IPs from all DMS reports.

There are two approaches for obtaining this. Either very simply add the IP addresses to the Sitecore.Analytics.ExcludeRobots.config as excludedIPAddresses.

But I did not like this approach. The customer editors are not robots, so this is messing up domain entities.. Instead I took a different approach and simply wrote a new processor for the startAnalytics pipeline.

## Excluding IP addresses from being tracked

First I wrote the processor which simply checks if the UserHostAddress of the current request exists in a static list of IP addresses.

```c#
  public class AnalyticsIpFilter
  {
    private static IPList _filteredIps;
    private static readonly object FilteredIpLock = new object();

    protected IPList FilteredIps
    {
      get
      {
        if (_filteredIps == null)
        {
          lock (FilteredIpLock)
            if (_filteredIps == null)
              _filteredIps = FilterIpRepository.Get();
        }
        return _filteredIps;
      }
    }

    public virtual void Process(PipelineArgs args)
    {
      Assert.ArgumentNotNull(args, "args");
      Assert.IsNotNull(HttpContext.Current, "HttpContext.Current");
      Assert.IsNotNull(HttpContext.Current.Request.UserHostAddress, "UserHostAddress");
      if (!AnalyticsSettings.Enabled)
        return;
      if (!RequestIpIsFiltered(IPAddress.Parse(HttpContext.Current.Request.UserHostAddress)))
        return;
      args.AbortPipeline();
    }

    private bool RequestIpIsFiltered(IPAddress userHostAddress)
    {
      return FilteredIps != null && FilteredIps.Contains(userHostAddress);
    }
  }
```

Then I wrote a repository which reads the excluded IP addresses from config using the built-in IPList class from the Sitecore API.

```c#
  public class FilterIpRepository
  {
    public static IPList Get()
    {
      var configNode = Factory.GetConfigNode("analytics/filterIps");
      if (configNode == null)
        return new IPList();
      return IPHelper.GetIPList(configNode) ?? new IPList();
    }
  }
```

The IPHelper.GetIPList(XmlNode) method expects a list of IP addresses or ranges separated by linebreaks or semi-colons.

Then I patched the IP addresses into config in a new .config file and patched the new processor in the beginning of the startAnalytics pipeline.

```xhtml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <pipelines>
      <startAnalytics>
        <processor patch:before="processor[@type='Sitecore.Analytics.Pipelines.StartAnalytics.Init,Sitecore.Analytics']" type="PT.DigitalMarketing.Infrastructure.AnalyticsIpFilter, PT.DigitalMarketing" />
      </startAnalytics>
	</pipelines>
    <analytics>
      <filterIps>
        127.0.0.1
      </filterIps>
    </analytics>
  </sitecore>
</configuration>
```

That was it, including sipping some coffee and waiting at the coffee machine, the actual implementation took less than an hour to complete. One might argue that it would have been faster just to add the IP addresses in the excludeRobots configuration but as I wrote in the beginning of the post, the editors are not robots and I don't like calling something a name which it isn't just because it is easier. In the long run this always leads to confusion and messy code.

Notice that this implementation does not work if you manually start the tracker elsewhere in code.

## Considerations

A completely different take on this task could have been to create a new traffic type in DMS called Internal which we then could mark all internal visits with so the customer could filter these visits out in their reports but still include them if desired.

This could be done by adding a new entry in the TrafficTypes table in the analytics database and then add a new processor in the trafficTypes pipeline.

This processor would then evaluate the IP in the same manner as shown in this post but instead of stopping the tracker then it would return the new custom TrafficType in the pipeline arguments.
