---
author: alc
category:
  - google-page-speed
  - modules
  - sitecore
cover:
  alt: Koala_cropped_1
  image: /wp-content/uploads/2014/09/Koala_cropped_1.jpg
date: "2014-09-08T11:03:28+00:00"
guid: http://laubplusco.net/?p=1016
title: Make Sitecore deliver images which fits the screen, part 2
url: /make-sitecore-deliver-images-fits-screen-part-2/

---
_This is the second part of a mini-series where I show how you can make Sitecore deliver images that fully support a responsive web design. [See the first post here if you haven't read it already.](/make-sitecore-deliver-images-fits-screen/ "Make Sitecore deliver images which fits the screen")_

In this part I will show how to make Sitecore center-crop images through the media handler using query string parameters.

The concept is basically that you call the media handler with specific querystring parameters that will make Sitecore center-crop the image. Similar to the out-of-the-box Sitecore image parameters.

The [previous post](/make-sitecore-deliver-images-fits-screen/ "Make Sitecore deliver images which fits the screen") in this series showed how to set JPEG compression level dynamically using a querystring parameter.

The whole point of these extra media features is to prevent unnecessary bandwidth use in responsive web design. JavaScript on the client can request Sitecore to deliver images in a compression ratio, resized and cropped to fit with the current screen view.

In a later bonus post I will show how this can be implemented client side.

## Why make yet another image cropper?

Several very fine modules already exists out there that can both crop and center crop images. For example [Image Cropper on MarketPlace](https://marketplace.sitecore.net/en/Modules/Image_Cropper.aspx) do exactly this.

The main reason for yet another cropping module is that I once again started coding before I googled the subject. It just seemed as an obvious thing to implement along with the JPEG compression processor.

The code that I show here also differ a bit from ImageCropper on some other key points.

1. The code does not use the normal  width and height parameters to crop the image. Instead there is separate querystring parameters for the crop width and height. This is done so Sitecores ResizeImageProcessor is not skipped. Using this approach you can  control the size of the image before cropping it to fit the screen.
1. The code does not extend or change the  sc:Image webcontrol. This is a design choice since the code is intended for supporting responsive web design and not something set server-side. The media handler is in this case supposed to be called from the client using JavaScript.
1. None of the existing getMediaStream processors are replaced or extended and the code does not rely on one big processor which can perform a large set of actions. Instead each action that can be performed on the image is performed in a specific separate processor with clear responsibility (in the end this is just an architectural preference and not related to the end functionality).

Lets just look at some code instead of me trying to explain myself.

## Cropping the image

Center-cropping an image in .NET using C# is quite straight forward when all input parameter validation is in place.

The following class center crops an image from a Stream and returns a Stream with the cropped image in the format that is passed to the method as a parameter. The code will center-crop any bitmap image format.

```c#
namespace PT.LaubPlusCo.ImageParameters.Infrastructure
{
  public class CropImageService
  {
    public static Stream CenterCrop(Stream imageStream, int width, int height, ImageFormat format)
    {
      var bitmap = new Bitmap(imageStream);
      if (bitmap.Width == width && bitmap.Height == height)
        return imageStream;

      var cropWidth = width < bitmap.Width && width != 0 ? width : bitmap.Width;
      var cropHeight = height < bitmap.Height && height != 0 ? height : bitmap.Height;
      var x = cropWidth == bitmap.Width ? 0 : (bitmap.Width / 2) - (cropWidth / 2);
      var y = cropHeight == bitmap.Width ? 0 : (bitmap.Height / 2) - (cropHeight / 2);

      var croppedBitmap = CropImage(bitmap, x, y, cropWidth, cropHeight);
      var memoryStream = new MemoryStream();
      croppedBitmap.Save(memoryStream, format);
      return memoryStream;
    }

    public static Bitmap CropImage(Bitmap originalImage, int x, int y, int width, int height)
    {
      if (x < 0 || ((x + width) > originalImage.Width) || (y < 0) || ((y + height) > originalImage.Height))
        return originalImage;
      var rectangle = new Rectangle(x, y, width, height);
      var croppedImage = new Bitmap(rectangle.Width, rectangle.Height);
      using (var g = Graphics.FromImage(croppedImage))
      {
        g.CompositingMode = CompositingMode.SourceCopy;
        g.DrawImage(originalImage, new Rectangle(0, 0, croppedImage.Width, croppedImage.Height), rectangle, GraphicsUnit.Pixel);
      }
      return croppedImage;
    }
  }
}
```

## Handling the media request

Now we can center crop the image so all we need to is simply make yet another getMediaStream pipeline processor to handle the media request.

In this processor we first check if the media request is for a "crop-able" image using the mimetype from the media item. Then we look for a query key indicating if we should crop the image.

If this key is present then we look for a crop-width and a crop-height key. These defaults to the actual image width or height if not set. If neither is set then we exit the processor since there is no cropping to perform. Otherwise, we do the cropping and set the new Stream as outputstream in the getMediaStream pipeline args.

```c#
namespace PT.LaubPlusCo.ImageParameters.Infrastructure
{
  public class CropImageProcessor
  {
    private string CropQueryKey
    {
      get { return Settings.GetSetting("LaubPlusCo.CropQueryKey", "c"); }
    }

    private string CropWidthQueryKey
    {
      get { return Settings.GetSetting("LaubPlusCo.CropWidthQueryKey", "cw"); }
    }

    private string CropHeightQueryKey
    {
      get { return Settings.GetSetting("LaubPlusCo.CropHeightQueryKey", "ch"); }
    }

    public IEnumerable<string> ValidMimeTypes
    {
      get
      {
        var validMimetypes = Settings.GetSetting("LaubPlusCo.CropValidMimeTypes", "image/jpeg|image/pjpeg|image/png|image/gif|image/tiff|image/bmp");
        return validMimetypes.Split(new[] {",", "|", ";"}, StringSplitOptions.RemoveEmptyEntries);
      }
    }

    public void Process(GetMediaStreamPipelineArgs args)
    {
      Assert.ArgumentNotNull(args, "args");
      if (args.Options.Thumbnail || !IsValidImageRequest(args.MediaData.MimeType))
        return;

      if (args.OutputStream == null || !args.OutputStream.AllowMemoryLoading)
        return;

      var cropKey = GetQueryOrCustomOption(CropQueryKey, args.Options.CustomOptions);
      if (string.IsNullOrEmpty(cropKey))
        return;

      var cropWidthOption = GetQueryOrCustomOption(CropWidthQueryKey, args.Options.CustomOptions);
      var cropHeightOption = GetQueryOrCustomOption(CropHeightQueryKey, args.Options.CustomOptions);

      if (string.IsNullOrEmpty(cropWidthOption) && string.IsNullOrEmpty(cropHeightOption))
        return;

      int cropWidth;
      if (!int.TryParse(cropWidthOption, out cropWidth))
        cropWidth = args.Options.Width;
      int cropHeight;
      if (!int.TryParse(cropHeightOption, out cropHeight))
        cropHeight = args.Options.Height;

      var croppedStream = CropImageService.CenterCrop(args.OutputStream.Stream, cropWidth, cropHeight, GetImageFormat(args.MediaData.MimeType.ToLower()));
      args.OutputStream = new MediaStream(croppedStream, args.MediaData.Extension, args.OutputStream.MediaItem);
    }

    private ImageFormat GetImageFormat(string mimeType)
    {
      switch (mimeType)
      {
        case "image/jpeg":
          return ImageFormat.Jpeg;
        case "image/pjpeg":
          return ImageFormat.Jpeg;
        case "image/png":
          return ImageFormat.Png;
        case "image/gif":
          return ImageFormat.Gif;
        case "image/tiff":
          return ImageFormat.Tiff;
          ;
        case "image/bmp":
          return ImageFormat.Bmp;
        default:
          return ImageFormat.Jpeg;
      }
    }

    protected bool IsValidImageRequest(string mimeType)
    {
      return ValidMimeTypes.Any(v => v.Equals(mimeType, StringComparison.InvariantCultureIgnoreCase));
    }

    protected string GetQueryOrCustomOption(string key, StringDictionary customOptions)
    {
      var value = WebUtil.GetQueryString(key);
      return string.IsNullOrEmpty(value) ? customOptions[key] : value;
    }
  }
}
```

Finally we insert the processor in config and add the various settings that the processor uses in an include config file.

```xhtml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <settings>
      <!-- Jpeg Compression level -->
      <setting name="LaubPlusCo.JpegCompressionLevelQueryKey" value="jq" />

      <!-- Image cropping -->
      <setting name="LaubPlusCo.CropQueryKey" value="c" />
      <setting name="LaubPlusCo.CropWidthQueryKey" value="cw" />
      <setting name="LaubPlusCo.CropHeightQueryKey" value="ch" />
      <setting name="LaubPlusCo.CropValidMimeTypes" value="image/jpeg|image/pjpeg|image/png|image/gif|image/tiff|image/bmp" />
    </settings>
    <pipelines>
      <getMediaStream>
        <processor patch:after="processor[@type='Sitecore.Resources.Media.GrayscaleProcessor, Sitecore.Kernel']"
            type="[NAMESPACE].CropImageProcessor, [Assembly]" />
        <processor type="[NAMESPACE].SetJpegCompressionProcessor, [Assembly]" />
      </getMediaStream>
    </pipelines>
  </sitecore>
</configuration>
```

 _The above config also contain the processor from the previous post._

Notice the order in  which the processors are patched in. We want Sitecore to resize or grayscale the image before we do anything. then we want to crop the image and as the last step we want to set the JPEG compression ratio.

Once again the Windows Koala will be my victim.

[![full-400](/wp-content/uploads/2014/09/full-400.png)](/wp-content/uploads/2014/09/full-400.png)

First the full image proportionally resized by standard Sitecore with height set to 400

[![koala_cropheight300](/wp-content/uploads/2014/09/koala_cropheight300.png)](/wp-content/uploads/2014/09/koala_cropheight300.png) Here we have the Koala center-cropped using a cropping height on 300 px.

[![koala_cropwidth300](/wp-content/uploads/2014/09/koala_cropwidth300.png)](/wp-content/uploads/2014/09/koala_cropwidth300.png) Here we have the Koala center-cropped using a cropping width on 300 px.

[![koalatimes75](/wp-content/uploads/2014/09/koalatimes75.png)](/wp-content/uploads/2014/09/koalatimes75.png)

Now we cropped down the Koala to height 75 (almost no Koala left)

[![koalacroppedascube](/wp-content/uploads/2014/09/koalacroppedascube.png)](/wp-content/uploads/2014/09/koalacroppedascube.png) And now cropped to a 125 x 125 centered cube still using height 400.

![graylowqualitykola](/wp-content/uploads/2014/09/graylowqualitykola.png)

Finally we have the Koala in a 200 x 200 cube using JPEG compression level 5 and even in grayscale using Sitecores pre-built gray query.

That was it, I hope someone find this useful.

## Where can I get my hands on this code precompiled?

I've tried uploading the module to Sitecore MarketPlace but they apparently have some issues at the moment.  I kept getting the Sitecore login screen when I was editing the module I'll try again soon and keep this post updated.

Until then simply copy-paste the code from this post.

Let me know if anybody want the code in a Github repo instead of MarketPlace then I'll create one.
