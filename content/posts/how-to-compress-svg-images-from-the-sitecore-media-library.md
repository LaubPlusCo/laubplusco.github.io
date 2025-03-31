---
author: alc
category:
  - google-page-speed
  - sitecore
  - tips
cover:
  alt: Gzip svg sitecore
  image: /wp-content/uploads/2014/09/20140901_170826-e1409584297782.jpg
date: "2014-09-01T15:08:04+00:00"
guid: http://laubplusco.net/?p=980
title: How to compress SVG images from the Sitecore media library
url: /compress-svg-images-sitecore-media-library/

---
This blog post is long overdue. The code was written some time last year when I started my still [on-going fight with Google Page Speed](/category/google-page-speed/).

When Sitecore deliver svg files from the media library there is no out-of-the-box approach for gzipping the media content and setting the Content-encoding response header to gzip.

## Gzipping the svg media content

First we need to gzip the media content.

To do this we insert the following processor into the end of the _getMediaStream_ pipeline.

```c#
public class CompressStream
{
  public void Process(GetMediaStreamPipelineArgs args)
  {
    Assert.ArgumentNotNull(args, "args");
    if (args.Options.Thumbnail)
      return;
    if (!IsCompressableService.IsCompressable(args.MediaData.Extension))
      return;
    var outputStream = args.OutputStream;
    var compressed = Compress(WriteBytesToStream(outputStream.Stream));
    var memoryStream = new MemoryStream(compressed);
    args.OutputStream = new MediaStream(memoryStream, args.MediaData.Extension, outputStream.MediaItem);
  }

  private static byte[] Compress(byte[] raw)
  {
    using (var memory = new MemoryStream())
    {
      using (var gzip = new GZipStream(memory, CompressionMode.Compress, true))
      {
        gzip.Write(raw, 0, raw.Length);
      }
      return memory.ToArray();
    }
  }

  private static byte[] WriteBytesToStream(Stream sourceStream)
  {
    using (var memoryStream = new MemoryStream())
    {
      sourceStream.CopyTo(memoryStream);
      return memoryStream.ToArray();
    }
  }
}
```

```xhtml
<sitecore>
  <pipelines>
    <getMediaStream>
      <processor type="[NAMESPACE].CompressStream, [ASSEMBLY].MediaCompress" />
    </getMediaStream>
  </pipelines>
</sitecore>
```

You will notice that we call a service class _IsCompressableService_ which checks if the media stream should be compressed.

```c#
  public class IsCompressableService
  {
    public static bool IsCompressable(string extension)
    {
      var settings = Settings.GetSetting("MediaCompress.CompressFileExtensions", "svg");
      var compressFileExtensions = settings.Split(new[] {",", "|", ";"}, StringSplitOptions.RemoveEmptyEntries);
      return compressFileExtensions.Any(extension.Contains);
    }
  }
```

This service reads a comma separated line from config:

```xhtml
<sitecore>
  <settings>
    <setting name="MediaCompress.CompressFileExtensions" value="svg" />
  </settings>
</sitecore>
```

Now the media stream is gzipped, so we need to tell the browsers that the content is gzip encoded so they can understand what it is we are sending them.

## Setting the Content-encoding response header

To do this we need to hook into the otherwise hidden event called media:request as I've also shown in this [post](/changing-a-pdf-on-the-fly-using-sitecore-media-request/ "Changing a PDF on the fly using Sitecore Media Request").

```c#
  public class MediaRequestEventHandler
  {
    public void OnMediaRequest(object sender, EventArgs args)
    {
      var sitecoreEventArgs = (SitecoreEventArgs) args;
      if (sitecoreEventArgs == null || !sitecoreEventArgs.Parameters.Any())
        return;
      var request = (MediaRequest) sitecoreEventArgs.Parameters[0];
      var media = MediaManager.GetMedia(request.MediaUri);
      if (!IsCompressableService.IsCompressable(media.Extension))
        return;
      SetContentEncoding(request.InnerRequest.RequestContext.HttpContext.Response);
    }

    private static void SetContentEncoding(HttpResponseBase response)
    {
      if (HasGzipContentEncoding(response.Headers))
        return;
      response.AddHeader("Content-encoding", "gzip");
    }

    private static bool HasGzipContentEncoding(NameValueCollection headers)
    {
      return headers.AllKeys.Any(k => k.Equals("content-encoding", StringComparison.InvariantCultureIgnoreCase))
        && headers["content-encoding"].Equals("gzip", StringComparison.InvariantCultureIgnoreCase);
    }
  }
```

We first get the media item which is being requested and then we check using the file extension if the content should be compressed. If so we add Content-encoding: gzip as a response header.

The configuration for hooking into the event looks like this.

```xhtml
<sitecore>
  <events>
    <event name="media:request">
      <handler type="[NAMESPACE].MediaRequestEventHandler, [ASSEMBLY].MediaCompress" method="OnMediaRequest" />
    </event>
  </events>
</sitecore>
```

That was it. The same code can gzip any other text based content such as js, css, xml etc. but it is better to use IIS dynamic compression for this. If anyone have a tip on how to configure IIS 7 / 7.5 to dynamic compress svg files please share as long as it doesn't imply changing the mime type to text/xml which is a terrible hack. It is possible to edit the applicationhost.config and set the mime-type here but this require you to have permissions (and time) to do this on all servers in the environment.

Please drop me a [mail](mailto:anders.laub@gmail.com) or a  comment if you would like me to share the code as a project or a MarketPlace module.
