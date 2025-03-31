---
author: alc
category:
  - asp.net
  - webapi
date: "2015-05-19T18:46:36+00:00"
guid: http://laubplusco.net/?p=1505
title: Gzip WebApi Response Content using an attribute
url: /gzip-webapi-response-content-using-an-attribute/

---
Today I was asked why json responses from WebApi was not being gzipped since it brought down our score on Google Page Speed.

So yet again I started my ongoing fight with Google Page Speed. It turns out that we had issues making IIS gzip application/json data. Since we do not have direct access to the production environment I decided that we would be better off simply gzipping the response content in code where and when we wanted to.

So I thought there must be an _**ActionFilterAttribute**_ for this already but it turns out there is not.

I searched and searched and came by this fine solution which adds a [MessageHandler that gzips responses](http://benfoster.io/blog/aspnet-web-api-compression).

But I would so much prefer just to be able to mark a method or a controller class with an attribute, that just seems cleaner.

So I decided to write the following ActionFilterAttribute :

```c#
using System;
using System.Linq;
using System.Web.Http.Filters;

namespace LaubPlusCo.WebApi.Compression
{
  [AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
  public class CompressFilter : ActionFilterAttribute
  {
    public override void OnActionExecuted(HttpActionExecutedContext context)
    {
      var acceptedEncoding = context.Response.RequestMessage.Headers.AcceptEncoding.First().Value;

      if (!acceptedEncoding.Equals("gzip", StringComparison.InvariantCultureIgnoreCase)
          && !acceptedEncoding.Equals("deflate", StringComparison.InvariantCultureIgnoreCase))
      {
        return;
      }

      context.Response.Content = new CompressedContent(context.Response.Content, acceptedEncoding);
    }
  }
}
```

Where _**CompressedContent**_ is a class that derives from _**HttpContent**_ highly inspired by the post I referenced above.

```c#
using System;
using System.IO;
using System.IO.Compression;
using System.Net;
using System.Net.Http;
using System.Threading.Tasks;

namespace LaubPlusCo.WebApi.Compression
{
  public class CompressedContent : HttpContent
  {
    private readonly string _encodingType;
    private readonly HttpContent _originalContent;

    public CompressedContent(HttpContent content, string encodingType = "gzip")
    {
      if (content == null)
      {
        throw new ArgumentNullException("content");
      }

      _originalContent = content;
      _encodingType = encodingType.ToLowerInvariant();

      foreach (var header in _originalContent.Headers)
      {
        Headers.TryAddWithoutValidation(header.Key, header.Value);
      }
      Headers.ContentEncoding.Add(encodingType);
    }

    protected override bool TryComputeLength(out long length)
    {
      length = -1;
      return false;
    }

    protected override Task SerializeToStreamAsync(Stream stream, TransportContext context)
    {
      Stream compressedStream = null;
      switch (_encodingType)
      {
        case "gzip":
          compressedStream = new GZipStream(stream, CompressionMode.Compress, true);
          break;
        case "deflate":
          compressedStream = new DeflateStream(stream, CompressionMode.Compress, true);
          break;
        default:
          compressedStream = stream;
          break;
      }

      return _originalContent.CopyToAsync(compressedStream).ContinueWith(tsk =>
      {
        if (compressedStream != null)
        {
          compressedStream.Dispose();
        }
      });
    }
  }
}
```

And now we can use the _**CompressFilter**_ attribute on classes like this:

```c#
[CompressFilter]
public class MyController : ApiController
{
  // ...
}
```

Or on methods like this:

```c#
public class MyController : ApiController
{
  [HttpGet]
  [CompressFilter]
  public IHttpActionResult Get()
  {
    //
  }
}
```

## When to use it?

It is best to let IIS perform the compression instead of doing it in code.

But if this for some reason is not possible to configure or if you for some reason only want selected responses to be gzipped then this attribute will do the job just perfect.

There is also a fine line for when it makes sense to let the server use time on gzipping the response. This is why IIS has an optional config attribute on the httpCompression element for setting a minimum file size before compression (minFileSizeForComp).

Hope this helps someone out there.
