---
author: alc
category:
  - asp.net
  - google-page-speed
  - modules
  - pipelines
  - sitecore
date: "2013-10-13T14:27:45+00:00"
guid: http://laubplusco.net/?p=46
title: Minify html in Sitecore
url: /minify-html-in-sitecore-pleasing-google-page-speed/

---
## **Pleasing Google Page Speed**

Minifying resources such as Javascript and CSS is becoming common practice nowadays and something which just should be done with no questions asked.

The reason for minification is both to reduce the payload and to reduce the number of requests made by the browser to the server. This makes the page load faster thus more responsive.

Google rewards pages with a quick load time in their search results as well which makes it one of the key points which are checked by their analysis tool called [Google Page Speed](https://developers.google.com/speed/pagespeed/ "Google Page Speed")

About two months ago we've just launched a new large Sitecore site for a customer at [Pentia](http://www.pentia.net "Pentia") where I've been the architect and lead developer on the solution. Just after go-live I went in and looked at the page speed results for the page.

I was not pleased with the result at first. The page only scored 62 / 100. On the list of suggestions for improving the results there was a bunch of quick fixes which I went through and the new result ended up on 90 / 100 which is quite fair, but still I wanted more.

Then I noticed one specific suggestion which I actually never really have given a thought before:

[https://developers.google.com/speed/docs/best-practices/payload#MinifyHTML](https://developers.google.com/speed/docs/best-practices/payload#MinifyHTML)

Minifying the html itself? Hm, why not give it a try

Minifying markup essentially means to removes all xml comments, all unnecessary whitespaces and line-breaks. Right-clicking on [Google.com](view-source:http://www.google.com) and viewing the source clearly shows how it looks.

## **Minify html in Sitecore**

To minify the markup I implemented a service class which uses regular expressions to remove all unwanted whitespaces, comments and linebreaks.

```c#
public static string Minify(string markup,
    bool removeWhitespaces = true, bool removeLineBreaks = true, bool removeHtmlComments = true)
  {
    if (removeWhitespaces)
      markup = RemoveWhiteSpaces(markup);
    if (removeLineBreaks)
      markup = RemoveLineBreaks(markup);
    if (removeHtmlComments)
      markup = RemoveHtmlComments(markup);
    return markup;
  }

  public static string RemoveHtmlComments(string markup)
  {
    return Regex.Replace(markup, "<!--*.*?-->", string.Empty, RegexOptions.Compiled | RegexOptions.Multiline);
  }

  public static string RemoveLineBreaks(string markup)
  {
    return Regex.Replace(markup, "\\n", string.Empty, RegexOptions.Compiled | RegexOptions.Multiline);
  }

  public static string RemoveWhiteSpaces(string markup)
  {
    return Regex.Replace(markup, "^\\s*", string.Empty, RegexOptions.Compiled | RegexOptions.Multiline);
  }
}
```

Okay now all the rendered markup just needs to go through the Minify method.

To do this I’ve implemented a HttpRequestProcessor which first checks if the current request should be minified and if so adds a custom Stream as Filter stream on the response.

```c#
public override void Process(HttpRequestArgs args)
  {
    if (!ShouldMinify())
      return;
    args.Context.Response.Filter = new MinifiedStream(args.Context.Response.Filter);
  }

  protected bool ShouldMinify()
  {
    if (!Sitecore.Configuration.Settings.GetBoolSetting("MarkupMinify.MinifyResponseMarkup", true))
      return false;
    if (args.Context.Request.Url.OriginalString.Contains("/sitecore")
      || args.Context.Request.Url.OriginalString.Contains("/speak"))
      return false;
    if (args.Context.Request.AcceptTypes == null || !args.Context.Request.AcceptTypes.Any(a => a.Equals("text/html", StringComparison.InvariantCultureIgnoreCase)))
      return false;
    return true;
  }
```

The first check reads a Sitecore setting in web.config checking if  minification is enabled at all.

```xhtml
<setting name="MarkupMinify.MinifyResponseMarkup" value="true" />
```

If it is then we’ll need to check if it makes sense to minify the current request. This is extremely important, we do not wish to minify the content editor or any Sitecore modules. For your solution it might make sense to adjust these rules.

The HttpRequestProcessor is then added in web.config to the httpRequestBegin pipeline as the last processor

```xhtml
<processor type="[NAMESPACE].MinifyMarkupProcessor, [ASSEMBLY]" />
```

The custom Stream called MinifiedStream wraps the Response.Stream.Filter in the property \_sink.

```c#
public class MinifiedStream : Stream
{
  private readonly StringBuilder _responseHtml;
  private readonly Stream _sink;

  public MinifiedStream(Stream sink)
  {
    _sink = sink;
    _responseHtml = new StringBuilder();
  }

  public override bool CanRead { get { return true; } }

  public override bool CanSeek { get { return true; } }

  public override bool CanWrite { get { return true; } }

  public override long Length { get { return 0; } }

  public override long Position { get; set; }

  public override void Flush()
  {
    _sink.Flush();
  }

  public override int Read(byte[] buffer, int offset, int count)
  {
    return _sink.Read(buffer, offset, count);
  }

  public override long Seek(long offset, SeekOrigin origin)
  {
    return _sink.Seek(offset, origin);
  }

  public override void SetLength(long value)
  {
    _sink.SetLength(value);
  }

  public override void Close()
  {
    _sink.Close();
  }
}
```

Each time something is written to the stream it gets appended to a StringBuilder. This is done until the end tag </html> appears and then the html is minified and written to the underlying stream.

```c#
  public override void Write(byte[] buffer, int offset, int count)
  {
    var strBuffer = Encoding.UTF8.GetString(buffer, offset, count);
    var eof = new Regex("</html>", RegexOptions.IgnoreCase);
    _responseHtml.Append(strBuffer);
    if (!eof.IsMatch(strBuffer))
      return;
    var html = _responseHtml.ToString();
    html = MinifyMarkupService.Minify(html);
    var data = Encoding.UTF8.GetBytes(html);
    _sink.Write(data, 0, data.Length);
  }
```

Well what if the end tag </html> never occurs? Well then nothing is written to the underlying stream thus resulting in an empty response. This “trick” is only for minifying whole pages of markup and not bits and pieces of a page. This is why the check ShouldMinify is extremely important.

Well how about performance then? I introduced this system on the site and it did not result in any performance decrease at all. I did not try to benchmark the actual minification process.

Did google Page Speed like it then? Yes indeed, the site front page went from the score 90/100 to 95/100. Then a week after Google changed Page Speed and the scoring system so now it is down on 91/100 which is also quite okay.

_**A small warning**_ , minifying the markup removes all xml comments which means that it is not possible to do [conditional commenting](http://msdn.microsoft.com/en-us/library/ms537512(VS.85).aspx) for detecting browser. This has to be done in Javascript instead.

I will add a project containing the code for this if requested, just write a comment.
