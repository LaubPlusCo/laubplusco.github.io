---
author: alc
category:
  - google-page-speed
  - sitecore
cover:
  alt: removing_exif_koala
  image: /wp-content/uploads/2014/09/removing_exif_koala.png
date: "2014-09-11T15:01:57+00:00"
guid: http://laubplusco.net/?p=1039
title: An involuntary lossless image compression by Sitecore
url: /involuntary-lossless-image-compression-sitecore/

---
_This is a small bonus post following my post series about [responsive images in Sitecore](/make-sitecore-deliver-images-fits-screen/ "Make Sitecore deliver images which fits the screen")._

When you load a JPEG image in .NET into a Bitmap and then draw it on another Bitmap then you will notice that the original image is sometimes bigger than the new one. How can that be..

This is because .NET does not "draw" the Exif information from the JPEG image onto the new Bitmap.

When the Sitecore _ResizeImageProcessor_ in the _getMediaStream_ pipeline resizes a JPEG image then it is actually being accidently lossless compressed since the Exif information is not stored and put back on to the image when it is resized.

## But this is really good stuff, right?

Yes it is. Google Page speed love it and therefore I love it.

If you run a large high quality photo site based on Sitecore then you will loose all the author information on the images when they are resized and in this onecase this unintended behavior is probably not good but for all the rest of us it is all joy and happy days.

## We want more..!!

What if we want Sitecore to always loss-less compress JPEG images just as the default behavior also for images which are delivered in full size.

Well here is how that is done in a primitive fashion where we simply imitate the involuntary behavior of the _ResizeImageProcessor_ (and my own processors from [my previous posts](/make-sitecore-deliver-images-fits-screen/ "Make Sitecore deliver images which fits the screen")).

_I should first note  that decoding and re-encoding an image might hurt the quality. I will dare to argue that the quality degradation in 99.999 % cases will not be visible at all for images shown on web pages. If image quality is paramount then do not use this approach._

The code is really simple. We start out with a new processor for the _getMediaStream_ pipeline.

```c#
namespace PT.LaubPlusCo.ImageParameters.Infrastructure
{
  public class RemoveExifProcessor
  {
    private bool RemoveExif
    {
      get { return Settings.GetBoolSetting("LaubPlusCo.RemoveExif", true); }
    }

    public void Process(GetMediaStreamPipelineArgs args)
    {
      Assert.ArgumentNotNull(args, "args");
      if (args.Options.Thumbnail || !IsJpegImageRequest(args.MediaData.MimeType) || !RemoveExif || RequestHasQueryString())
        return;
      var losslessStream = RemoveExifService.Remove(args.OutputStream.Stream);
      args.OutputStream = new MediaStream(losslessStream, args.MediaData.Extension, args.OutputStream.MediaItem);
    }

    private bool RequestHasQueryString()
    {
      return !string.IsNullOrEmpty(WebUtil.GetQueryString());
    }

    protected bool IsJpegImageRequest(string mimeType)
    {
      return mimeType.Equals("image/jpeg", StringComparison.InvariantCultureIgnoreCase)
             || mimeType.Equals("image/pjpeg", StringComparison.InvariantCultureIgnoreCase);
    }
  }
}
```

To prevent unnecessary resource usage we perform a pragmatic check on the querystring to see if it contains anything. Presuming that values in the querystring anyway would result in a re-encoding of the image. This check could instead look specifically for query keys which triggers an image re-encoding such as w, h, etc.

Then we write our service class which creates a bitmap from the media Stream and then saves the bitmap in a MemoryStream.

```c#
namespace PT.LaubPlusCo.ImageParameters.Infrastructure
{
  public class RemoveExifService
  {
    public static Stream Remove(Stream imageStream)
    {
      var originalImage = new Bitmap(imageStream);
      var rectangle = new Rectangle(0, 0, originalImage.Width, originalImage.Height);
      var losslessCompressedImage = new Bitmap(rectangle.Width, rectangle.Height);
      using (var g = Graphics.FromImage(losslessCompressedImage))
      {
        g.CompositingMode = CompositingMode.SourceCopy;
        g.DrawImage(originalImage, new Rectangle(0, 0, originalImage.Width, originalImage.Height), rectangle, GraphicsUnit.Pixel);
      }
      var memoryStream = new MemoryStream();
      losslessCompressedImage.Save(memoryStream, ImageFormat.Jpeg);
      return memoryStream;
    }
  }
}
```

Finally we write the include configuration.

```xhtml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <settings>
      <setting name="LaubPlusCo.RemoveExif" value="true" />
    </settings>
    <pipelines>
      <getMediaStream>
        <processor patch:before="processor[@type='Sitecore.Resources.Media.ThumbnailProcessor, Sitecore.Kernel']"
            type="PT.LaubPlusCo.ImageParameters.Infrastructure.RemoveExifProcessor, PT.LaubPlusCo.ImageParameters" />
      </getMediaStream>
    </pipelines>
  </sitecore>
</configuration>
```

As always Sitecore caches the output of the getMediaStream pipeline so the resource consumption is not considered. This is of course under the presumption that Sitecore media cache is not disabled or for some weird reason unreliable.

That was it. I hope some day that Sitecore will make it a standard setting or even better a standard Site setting if all images from the media lib should be losslessly compressed. PNG's as well.
