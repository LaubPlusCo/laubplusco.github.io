---
author: alc
category:
  - best-practices
  - configuration
  - sitecore
date: "2014-01-08T11:43:30+00:00"
guid: http://laubplusco.net/?p=545
title: Using threshold warning limits in Sitecore
url: /setting-threshold-warning-limits-sitecore/

---
When working on an old Sitecore solution the log files are almost always completely spammed with different threshold warnings.

744 10:28:14 WARN Item threshold exceeded for web page. Items accessed: 2578. Threshold: 2000. Page URL: /xxxx.aspx
7672 10:28:15 WARN Item threshold exceeded for web page. Items accessed: 29145. Threshold: 2000. Page URL:xxxx.aspx
7672 10:28:15 WARN Memory threshold exceeded for web page. Memory used: 37813KB. Threshold: 35000KB. Page URL:xxxx.aspx

This is of course if you have not changed the threshold warning limits in configuration or turned off the threshold warnings completely which is only possible from version 6.4.

Even Sitecore 7.1 ships with the same extremely low threshold limits as 6.0 which is almost the same as they were back in Sitecore 5.3 except for the MemoryMonitorHook which has been increased to 800MB where it was 500MB in 5.3:

```xhtml
      <httpRequestEnd>
		...
        <processor type="Sitecore.Pipelines.HttpRequest.StopMeasurements, Sitecore.Kernel">
          <ShowThresholdWarnings>false</ShowThresholdWarnings>
          <TimingThreshold desc="Milliseconds">1000</TimingThreshold>
          <ItemThreshold desc="Item count">1000</ItemThreshold>
          <MemoryThreshold desc="KB">10000</MemoryThreshold>
        </processor>
      </httpRequestEnd>

...
    <hooks>
      <hook type="Sitecore.Diagnostics.HealthMonitorHook, Sitecore.Kernel" />
      <hook type="Sitecore.Diagnostics.MemoryMonitorHook, Sitecore.Kernel">
        <param desc="Threshold">800MB</param>
        <param desc="Check interval">00:00:05</param>
        <param desc="Minimum time between log entries">00:01:00</param>
        <ClearCaches>false</ClearCaches>
        <GarbageCollect>false</GarbageCollect>
        <AdjustLoadFactor>false</AdjustLoadFactor>
      </hook>
    </hooks>
```

In Sitecore versions prior to 6.4 it was not possible to turn off the log threshold warnings by setting ShowThresholdWarning to false on the StopMeasurements processor.

So what Sitecore actually did in version 6.4 to give us cleaner log files was simply to disable the threshold logging.

It keeps me somewhat happy but it still seems a bit like peeing in the pants to keep you warm.

The threshold limits can be quite useful for diagnosing the state of a solution and now they are just taking up space in the web.config on most solutions.

To get any value out of the threshold warnings then the completely outdated limits needs to be changed.

You would of course always like your worker process to use a whole lot more memory than 800 MB and each request to be able to use more than 10000KB. What should the limits be set to then to reduce the spamming of log files but still give some valuable indications in the log files?

This completely depends on the memory and other resources available in your environment and if it is a 32 or 64 bit system etc.

For example if 4GB is available on a x64 server then 2800MB would be a fine setting for the memory hook if the web server is only running this one site.

The most valuable threshold warning is the ItemThreshold which reacts on the amount of items read by a single request.

This threshold can really indicate if some developer is traversing all descendants of an item (don't ever) or messing up some recursion which impacts the performance of the page. The limit should be set to more than a 1000 items though to provide any real value, again this is solution dependent, especially when using datasources on renderings then 1000 is actually a low number.

My recommendation is to set ShowThresholdWarnings to true and adjust the limits to fit with the production environment, play a bit around with the values so it fits with your setup.

If the log files then starts to be spammed with threshold warnings again then figure out why your limits are reached rather than just turning the warnings off.

_A lot of blog posts have been written about the threshold warnings before 6.4 was released but then it seems like everyone has forgotten about them again. Me as well, I was just reminded now when debugging on an old solution._
