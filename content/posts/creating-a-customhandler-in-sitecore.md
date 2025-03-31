---
author: alc
category:
  - configuration
  - sitecore
  - tips
date: "2013-10-22T14:52:11+00:00"
guid: http://laubplusco.net/?p=193
title: Creating a customHandler in Sitecore
url: /creating-a-customhandler-in-sitecore/

---
Sometimes you would like all requests to a specific path and below to be handled by a specific generic handler. For example in the same manner as the mediaHandler works in Sitecore.

## Setting up a customHandler in Sitecore

Configuring a normal httpHandler in .NET is easy, you simply add entries in the web.config depending on your version of IIS.

Under /configuration/system.webServer/handlers ( [for IIS 7 and above](http://www.iis.net/configreference/system.webserver/handlers))

```xhtml
<add verb="*" path="examplehandler.ashx" type="[NAMESPACE].ExampleHandler, [ASSEMBLY]" name="ExampleHandler" />
```

and the same under /configuration/system.web/httpHandlers (for older versions of IIS)

```c#
<add verb="*" path="examplehandler.ashx" type="[NAMESPACE].ExampleHandler, [ASSEMBLY]" name="ExampleHandler" />
```

To make the handler react on a specific path simply add a path attribute in the configuration and all requests which contains this path will be handled by this specific generic handler. An example could be:

```xhtml
<add verb="*" path="*.example" type="[NAMESPACE].ExampleHandler, [ASSEMBLY]" name="ExampleHandler" />
```

Where the ExampleHandler class will handle all requests for any path postfixed with .example

Sitecore made a small addition to this configuration enabling the handlers to be resolved by Sitecore.

To make this work do not add a path attribute to the handlers as you normally would in ASP.NET instead create a new config entry as follows under /configuration/sitecore/customHandlers/ see the last entry in the below example:

```xhtml
    <customHandlers>
      <handler trigger="~/media/" handler="sitecore_media.ashx" />
      <handler trigger="~/api/" handler="sitecore_api.ashx" />
      <handler trigger="~/xaml/" handler="sitecore_xaml.ashx" />
      <handler trigger="~/icon/" handler="sitecore_icon.ashx" />
      <handler trigger="~/feed/" handler="sitecore_feed.ashx" />
      <handler trigger="/-/examplehandler/" handler="examplehandler.ashx" />
    </customHandlers>
```

Now the handler we defined before called examplehandler.ashx will catch all requests to the relative path /-/examplehandler/ as start path and handle the request with the specified handler.

Now to implementing the handler.

```c#
public class ExampleHandler: IHttpHandler, IRequiresSessionState
{
  public void ProcessRequest(HttpContext context)
  {
    context.Response.Write("Hello world");
  }

  public bool IsReusable { get; private set; }
}
```

Remember to implement the IRequiresSessionState interface if the handler should use the ASP.NET session.

That is it. In a soon to come blog post I will show how this concept can be used to deliver CSS dynamically generated from items in Sitecore using LESS.

I dare you to like this post..
