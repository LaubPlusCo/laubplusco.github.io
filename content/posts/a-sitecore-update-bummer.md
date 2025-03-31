---
author: alc
category:
  - errors
  - sitecore
  - sitecore-6.6
  - sitecore-updates
date: "2013-10-13T19:31:21+00:00"
guid: http://laubplusco.net/?p=109
title: A Sitecore update bummer
url: /sitecore-update-bummer/

---
When update 6.6 rev 130111 was released I upgraded a customer solution from the previous 6.6 release thinking that this was only a minor update which would solve some of the other issues the previous revision have had. I was terrible wrong and the upgrade led me into a ton of trouble.

The first issue in the new revision was an added processor in the renderField pipeline.

```xhtml
      <renderField>
...
        <processor type="Sitecore.Pipelines.RenderField.GetTextFieldValue, Sitecore.Kernel" />
...
      </renderField>
```

This processor ensures that all text fields and multi line text fields is output escaped when rendered through the renderField pipeline.

From reflector, notice the HttpUtility.HtmlEncode call.

```c#
  public class GetTextFieldValue
  {
    public void Process(RenderFieldArgs args)
    {
      Assert.ArgumentNotNull((object) args, "args");
      string fieldTypeKey = args.FieldTypeKey;
      if (fieldTypeKey != "text" && fieldTypeKey != "single-line text")
        return;
      args.WebEditParameters.Add("prevent-line-break", "true");
      args.Result.FirstPart = HttpUtility.HtmlEncode(args.Result.FirstPart);
    }
  }
```

Well this is actually a really good security mechanism.

That is if the text field value could come from some malicious Sitecore editor.

But does it make sense to protect Sitecore from its editors? I think not, they could just use a rich text field to go about their evil doings instead.

Even though it is extremely bad practice you'll often see solutions where either experienced editors or lazy developers have added markup to some text field  especially in older Sitecore solution this has been almost common practice.

Now this markup renders output escaped on the upgraded solutions looking kind of funny.

Back when using xslt's was "cool" (it really never was, never ever again) in Sitecore 5.x then writing the value of a field could be done like this:

```xhtml
<xsl:value-of select="sc:fld('My Field',.)" disable-output-escaping="yes" />
```

Where it was actually easy to explicitly tell whether or not the field value should be output escaped.

Even though I hate writing this, then that actually made a whole lot more sense than the new GetTextFieldValue processor. Let the developers decide whether or not the field should be output escaped. Please do not do it automatically without us having the option of stopping it. We are not completely brain dead. So please Sitecore add it as an attribute on the sc:Text webcontrol instead so we can control the output escaping.

Something is really wrong when functionality in xslt seems better.

To stop the output escaping, either simply remove the processor from the renderField pipeline or implement your own version which do not call the HttpUtility.HtmlEncode but only does the prevent-line-breaks stuff. This is a bit nasty but it works just fine.

The update also contained the AntiCsrf module which led to a ton of other trouble. More about this in another blog post.

I cannot really see why this update was deemed to be an update and not a new minor version so us poor hard working developers would know that it contained some new breaking changes.

Both of these things are now included in Sitecore 7. So from now on we just need to be aware of their existence, learning the hard way on this one.
