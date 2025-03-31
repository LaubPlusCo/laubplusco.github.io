---
author: alc
category:
  - asp.net
  - google-page-speed
  - tips
cover:
  alt: Gzip svg sitecore
  image: /wp-content/uploads/2014/09/20140901_170826-e1409584297782.jpg
date: "2014-09-03T12:40:55+00:00"
guid: http://laubplusco.net/?p=990
title: How to gzip svg files in ASP.NET
url: /gzip-svg-files-asp-net/

---
_My [previous pos](/compress-svg-images-sitecore-media-library/ "How to compress SVG images from the Sitecore media library") t showed how to gzip svg files served by the Sitecore media library. This is fine if all svg files reside in the media library but this is typical not the case._

In this post I will show how to gzip compress any svg files served by your ASP.NET solution. As I explained in my previous post the issue that I am attempting to solve is that IIS will not by default gzip svg files without a lot of configuration changes on server level.

The solution here is extremely simple and pragmatic. Simply implement a HttpModule and set the Response.Filter stream to a GZipStream.

## GZip svg files

```c#
  public class SvgCompressionModule : IHttpModule
  {
    public void Init(HttpApplication application)
    {
      application.BeginRequest += ApplicationOnBeginRequest;
    }

    public void Dispose()
    {
    }

    private void ApplicationOnBeginRequest(object sender, EventArgs eventArgs)
    {
      var app = (HttpApplication) sender;
      if (app == null)
        return;
      var context = app.Context;
      if (!IsSvgRequest(context.Request))
        return;
      context.Response.Filter = new GZipStream(context.Response.Filter, CompressionMode.Compress);
      context.Response.AddHeader("Content-encoding", "gzip");
    }

    protected virtual bool IsSvgRequest(HttpRequest request)
    {
      var path = request.Url.AbsolutePath;
      return Path.HasExtension(path) &&
             Path.GetExtension(path).Equals(".svg", StringComparison.InvariantCultureIgnoreCase);
    }
  }
```

Then add the httpmodule to configuration

```xhtml
<configuration>
  ...
  <system.webServer>
    <modules>
      ...
      <add name="SvgCompressionModule" type="[NAMESPACE].SvgCompressionModule, [ASSEMBLY].SvgCompression" />
    </modules>
  </system.webServer>
  ...
  <system.web>
    <httpModules>
      ...
      <add name="SvgCompressionModule" type="[NAMESPACE].SvgCompressionModule, [ASSEMBLY].SvgCompression" />
    </httpModules>
  </system.web>
</configuration>
```

That is it. I hope this helps someone out there in their own personal fight with Google PageSpeed.
