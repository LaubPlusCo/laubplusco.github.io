---
author: alc
category:
  - best-practices
  - pipelines
  - sitecore
  - tips
date: "2013-10-13T18:04:06+00:00"
guid: http://laubplusco.net/?p=93
title: A note about Sitecore pipelines
url: /sitecore-pipelines/

---
My two previous posts both contained implementations  of custom processors for the httpRequestBegin pipeline.

Just a quick explanation about their structure and some absolute best practices when implementing your own Sitecore pipeline processors.

## Check and exit fast

Since the httpRequestBegin pipeline handles all requests it is extremely important to check the context for validity before continuing processing. If a simple check fails then exit the processor right away.

_**Example**_

```c#
public class MyCustomProcessor
{
  public void Process(PipelineArgs args)
  {
    if (!IsContextValidForProcessor())
      return;
    DoSomeOperation();
  }
}
```

This rule applies for all pipeline processors, if the context is not valid for the operation exit immediately and let the rest of the pipeline continue on.

## Make least expensive checks first

One check might be more expensive than the others and to prevent it from being unnecessary executed in a context where the pipeline processor should not run anyway, always perform the least expensive checks first.

_**Example**_

```c#
public class MyCustomProcessor
{
  public void Process(PipelineArgs args)
  {
    if (Context.Site == null)
      return;
    if (Context.Site.Name.Equals(Sitecore.Constants.ShellSiteName, StringComparison.InvariantCultureIgnoreCase))
      return;
    var requiredValueFromItem = ReadSomeValueFromItem();
    if (requiredValueFromItem == null)
      return;
    DoSomeOperation();
  }
}
```

String comparison is cheaper than reading database values. The above example check is a very typical example checking if the current site context is the shell site.

That was it for this post.
