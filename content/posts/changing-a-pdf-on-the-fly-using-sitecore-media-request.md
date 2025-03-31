---
author: alc
category:
  - events
  - media
  - modules
  - sitecore
date: "2013-10-23T20:50:21+00:00"
guid: http://laubplusco.net/?p=213
title: Changing a PDF on the fly using Sitecore Media Request
url: /changing-a-pdf-on-the-fly-using-sitecore-media-request/

---
Some time ago I had a fun challenge for a customer. They wanted a legal notice pdf to be appended to all PDF files which was downloaded from specific parts of the media library on the fly.

They also wanted to be able to update the legal notice pdf and use a different one for different parts of the media library.

The same concept could also be used for adding a watermark to a pdf or maybe removing all above the first three pages for non-authenticated users etc.

For altering the PDF's I use [WebSuperGoo's ABCpdf8](http://www.websupergoo.com/abcpdf-8.htm).

## Handling the Sitecore Media Request

First thing is to catch all media requests using the Sitecore media request event, media:request.

This event is NOT in a normal out-of-the-box web.config but you can add it yourself as shown below.

```c#
      <event name="media:request">
        <handler type="[NAMESPACE].AppendLegalNoticeEventHandler, [ASSEMBLY]" method="OnMediaRequest" />
      </event>
```

Create the class..

```c#
public class AppendLegalNoticeEventHandler
{
  public void OnMediaRequest(object sender, EventArgs args)
  {

  }
}
```

Next thing to do is to determine if the request is for a pdf file in a specific part of the media library (using the media items path) if not exit quickly otherwise append the relevant pdf to the start of the requested pdf document.

```c#
public class AppendLegalNoticeEventHandler
{
  private const string PdfMimeType = "application/pdf";
  private const string PdfExtension = "pdf";

  public void OnMediaRequest(object sender, EventArgs args)
  {
    if (Context.Site.Name.Equals(Sitecore.Constants.ShellSiteName, StringComparison.InvariantCultureIgnoreCase))
      return;
    var sitecoreEventArgs = (SitecoreEventArgs)args;
    if (sitecoreEventArgs == null || !sitecoreEventArgs.Parameters.Any())
      return;
    var request = (MediaRequest)sitecoreEventArgs.Parameters[0];
    if (request == null || ResponseBeingRedirected(request))
      return;
    var media = MediaManager.GetMedia(request.MediaUri);
    if (!IsPdfRequest(media))
      return;
    var legalNoticeSettings = LegalNoticeSettingsRepository.Get();
    if (legalNoticeSettings == null)
      return;
    var legalNoticeArea = GetLegalNoticeArea(legalNoticeSettings, media);
    if (legalNoticeArea == null || legalNoticeArea.PdfFile == null)
      return;
    WriteFileToResponse(AppendLegalNotice(legalNoticeArea, media), request);
  }

  private static bool ResponseBeingRedirected(MediaRequest request)
  {
    var response = request.InnerRequest.RequestContext.HttpContext.Response;
    return response.IsRequestBeingRedirected;
  }

  private static void WriteFileToResponse(byte[] pdfData, MediaRequest request)
  {
    var response = request.InnerRequest.RequestContext.HttpContext.Response;
    response.OutputStream.Write(pdfData, 0, pdfData.Length);
    response.StatusCode = (int)HttpStatusCode.OK;
    response.ContentType = PdfMimeType;
    response.Flush();
    response.End();
  }

  private static byte[] AppendLegalNotice(LegalNoticeArea legalNoticeArea, Media media)
  {
    var originalDocument = new Doc();
    originalDocument.Read(media.GetStream().Stream);
    var notice = new Doc();
    notice.Read(legalNoticeArea.PdfFile.GetMediaStream());
    notice.Append(originalDocument);
    return notice.GetData();
  }

  private LegalNoticeArea GetLegalNoticeArea(LegalNoticeSettings legalNoticeSettings, Media media)
  {
    return legalNoticeSettings.Areas.FirstOrDefault(a => media.MediaData.MediaItem.InnerItem.Paths.FullPath.StartsWith(a.MediaLibraryPath, StringComparison.InvariantCultureIgnoreCase));
  }

  private static bool IsPdfRequest(Media media)
  {
    return media.Extension.Equals(PdfExtension, StringComparison.InvariantCultureIgnoreCase) || media.MimeType.Equals(PdfMimeType, StringComparison.InvariantCultureIgnoreCase);
  }
}
```

The LegalNoticeArea class here is just a carrier class for a pdf file and a string for the root path of the media library for which this pdf has to be added.

```c#
public class LegalNoticeArea
{
  public MediaItem PdfFile { get; set; }
  public string MediaLibraryPath { get; set; }
}
```

The LegalNoticeSettings is a settings item retrived through the link database. I will not show the implementation of this now.

The code is extremely fast, but it still it adds about 1-200 milliseconds for the pdf to start downloading. This should be okay though, since the users will accept a slower response time just fine when clicking a download button compared to when they are clicking a link. Of course it take up some server resources which should be considered if pdf files are downloaded several times a second. Also make sure to cache the settings item somehow or read it in a quick manner so it does not slow up the request unnecessary.

One might argue that the best solution would be to append/modify the pdf before it was stored in the media cache but then the media cache would have to be cleared as soon as the editors alters the pdf to be appended and this does not seem so smart an idea after all.

Feel free to comment or let me know if I should post the whole code in a zipped project.
