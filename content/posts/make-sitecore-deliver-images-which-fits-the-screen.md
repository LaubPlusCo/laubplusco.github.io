---
author: alc
category:
  - google-page-speed
  - modules
  - sitecore
cover:
  alt: JPEG quality compression Sitecore
  image: /wp-content/uploads/2014/09/koala_top.png
date: "2014-09-04T19:23:23+00:00"
guid: http://laubplusco.net/?p=1000
title: Make Sitecore deliver images which fits the screen
url: /make-sitecore-deliver-images-fits-screen/

---
When implementing a responsive design the size of images is almost always an issue. You want the images to be resized server-side so they fit the screen. Primarily for performance but also to please Google Page Speed.

Sitecore enables us to set the width and height dynamically by sending query parameters to the media handler.

My colleague Brian [wrote this nice post where he explains the basic out-of-the-box Sitecore image parameters](http://briancaos.wordpress.com/2011/02/28/sitecore-image-parameters/).

But what if it was possible to also let say set the JPEG compression level using query parameters? Then it would be possible to set the compression level low for small screens and high on desktops. And what about center-cropping images to make them fit in those designs where the image proportions differ between breakpoints.

Wait no longer. This post will show how to set the JPEG compression level using a query parameter and my next post in this mini-series will show how to crop images using a similar technique.

What about performance then? It can take up a lot of resources on the server when working with images.

Well  when changing the media in the right pipeline Sitecore will do its magic and store the image in the media cache.

## Setting JPEG compression level via the query string

To do this we implement a service class which changes the compression level of a Stream. Plain and simple .NET

```c#
  public class ChangeJpegCompressionLevelService
  {
    public static Stream Change(MediaStream mediaStream, int jpegQuality)
    {
      var jpegEncoder = GetEncoder(ImageFormat.Jpeg);
      var qualityEncoder = Encoder.Quality;
      var encoderParameters = new EncoderParameters(1);
      var bitmap = new Bitmap(mediaStream.Stream);
      encoderParameters.Param[0] = new EncoderParameter(qualityEncoder, jpegQuality);
      var stream = new MemoryStream();
      bitmap.Save(stream, jpegEncoder, encoderParameters);
      return stream;
    }

    private static ImageCodecInfo GetEncoder(ImageFormat format)
    {
      var codecs = ImageCodecInfo.GetImageDecoders();
      return codecs.FirstOrDefault(codec => codec.FormatID == format.Guid);
    }
  }
```

Next thing we create a new processor for the getMediaStream pipeline.

```c#
namespace PT.LaubPlusCo.ImageParameters.Infrastructure
{
  public class SetJpegCompressionProcessor
  {
    private string JpegCompressionLevelQueryKey
    {
      get { return Settings.GetSetting("LaubPlusCo.JpegCompressionLevelQueryKey", "jq"); }
    }

    public void Process(GetMediaStreamPipelineArgs args)
    {
      Assert.ArgumentNotNull(args, "args");
      if (args.Options.Thumbnail || !IsJpegImageRequest(args.MediaData.MimeType))
        return;

      if (args.OutputStream == null || !args.OutputStream.AllowMemoryLoading)
        return;

      var jpegQualityQuery = WebUtil.GetQueryString(JpegCompressionLevelQueryKey);
      if (string.IsNullOrEmpty(jpegQualityQuery))
        return;

      int jpegQuality;
      if (!int.TryParse(jpegQualityQuery, out jpegQuality) || jpegQuality <= 0 || jpegQuality > 100)
        return;

      var compressedStream = ChangeJpegCompressionLevelService.Change(args.OutputStream, jpegQuality);
      args.OutputStream = new MediaStream(compressedStream, args.MediaData.Extension, args.OutputStream.MediaItem);
    }

    protected bool IsJpegImageRequest(string mimeType)
    {
      return mimeType.Equals("image/jpeg", StringComparison.InvariantCultureIgnoreCase)
             || mimeType.Equals("image/pjpeg", StringComparison.InvariantCultureIgnoreCase);
    }
  }
}
```

We first check if it is a jpeg image which is being requested and if the query key is present and contain a valid compression level between 1 and 100.

Finally we call the service class with the image stream and replace the media stream in the pipeline args with the new one.

The processor should run after Sitecore have performed resizing etc. so we place it in the end of the getMediaStream pipeline using the following include config.

```xhtml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <settings>
      <setting name="LaubPlusCo.JpegCompressionLevelQueryKey" value="jq" />
    </settings>
    <pipelines>
      <getMediaStream>
        <processor patch:after="processor[@type='Sitecore.Resources.Media.GrayscaleProcessor, Sitecore.Kernel']"
          type="PT.LaubPlusCo.ImageParameters.Infrastructure.SetJpegCompressionProcessor, PT.LaubPlusCo.ImageParameters" />
      </getMediaStream>
    </pipelines>
  </sitecore>
</configuration>
```

That is it. Now we can set the JPEG compression level by adding the query jq=<quality> and even mix this with the other Sitecore image parameters, for example:

http://MYWEBSITE/~/media/my-image.jpg?w=100&h=100&jq=60

I have used the well-known Windows Koala as a show case

[![koala_low_compression](/wp-content/uploads/2014/09/koala_low_compression.png)](/wp-content/uploads/2014/09/koala_low_compression.png)_Low compression_ [![koala_medium_compression](/wp-content/uploads/2014/09/koala_medium_compression.png)](/wp-content/uploads/2014/09/koala_medium_compression.png) _Medium compression_ [![koala_high_compression](/wp-content/uploads/2014/09/koala_high_compression.png)](/wp-content/uploads/2014/09/koala_high_compression.png) _High compression_

What about caching then? Well Sitecore has already taken care of this all by itself.

[![koala_mediacache](/wp-content/uploads/2014/09/koala_mediacache.png)](/wp-content/uploads/2014/09/koala_mediacache.png)_Stay tuned for my [next post which shows how to center-crop images using query string values](/make-sitecore-deliver-images-fits-screen-part-2/ "Make Sitecore deliver images which fits the screen, part 2")._
