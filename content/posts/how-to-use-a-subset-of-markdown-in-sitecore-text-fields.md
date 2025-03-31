---
author: alc
category:
  - modules
  - sitecore
cover:
  alt: markdown_preview
  image: /wp-content/uploads/2014/08/markdown_preview.png
date: "2014-08-17T18:57:21+00:00"
guid: http://laubplusco.net/?p=937
title: How to use a subset of markdown in Sitecore text fields
url: /use-subset-markdown-sitecore-text-fields/

---
_This is my first blog post for some months now so I'll start somewhat easy._

I attended a design presentation meeting this Friday where the customer presented a nice and simple design based on bootstrap for their new website.

All was great about the design but then I noticed something funny about the spot headlines used in the design. The headlines looked somewhat like this:

## This is what **a headline** could look like

_Notice how the headline uses a slim text whilst some of the words are bold, also notice the irregular line-break._

A field for a headline is usually just a single-line text field and we do not want the editors to include "handwritten" markup in other than rich-text fields. A solution could be to make the headlines for the spots into rich-text field with a custom profile that only shows the bold button. Knowing how editors behave and how the rich-text editor would insert tons of garbage as default this solution would be a potential nightmare of customization.

## What about Markdown?

I've become quite fond of the [Markdown](http://en.wikipedia.org/wiki/Markdown "Wiki Markdown") syntax lately, so much that I am even contemplating re-making my blog in [Ghost](https://ghost.org/ "Ghost blog engine")(I also fell in love with node.js). So after dismissing the idea about using rich-text fields for headlines I thought about introducing a small subset of the Markdown syntax to be used in Sitecore single-line text fields.

Some quick googling lead me to [Sitecore Markdown Field](https://github.com/Sitecore/Sitecore-MarkdownField) which is a nice project that introduce a whole new Markdown field type for Sitecore. This is really nice but also going a bit too far since we only need a small subset of the syntax to work on selected text fields and not the whole lot with a custom field.

## The solution

The solution is actually extremely simple. We need to support the syntax for bold, italic and linebreaks only in specific text fields.

First we create a new processor for the renderField pipeline.

```c#
namespace LaubPlusCo.MarkDown.Infrastructure
{
  public class GetMarkDownTextValue
  {
    public void Process(RenderFieldArgs args)
    {
      Assert.ArgumentNotNull(args, "args");
      var fieldTypeKey = args.FieldTypeKey;
      if (fieldTypeKey != "text" && fieldTypeKey != "single-line text" && IsFieldMarkedForMarkDown(args.GetField()))
        return;

      // Note, Sitecore encodes single line text fields since v6.6 (shame on Sitecore)
      // so we need to decode it back first.
      args.Result.FirstPart = HttpUtility.HtmlDecode(args.Result.FirstPart);

      if (Context.PageMode.IsPageEditorEditing)
        return;

      args.Result.FirstPart = MarkDownTextService.Markdown(args.Result.FirstPart);
    }

    private bool IsFieldMarkedForMarkDown(Field field)
    {
      var markedForMarkdownFields = MarkDownFieldConfiguration.GetMarkDownFieldNames();
      return markedForMarkdownFields.Any(c => c.Equals(field.ID.ToString(), StringComparison.InvariantCultureIgnoreCase)
                                              || c.Equals(field.Name, StringComparison.InvariantCultureIgnoreCase));
    }
  }
}
```

In this processor we first checks if the current field is a text field, if it is in the list of fields to _markdownify_ and finally if the page is not in page edit mode. If everything is good the result of the renderfield pipeline should return the text _markdownified_.

_Note that Sitecore started output escaping both multi and single line text fields back in v6.6, see [this post](/sitecore-update-bummer/ "A Sitecore update bummer") for more grumbling._

Next we implement the Service class which performs the _markdownification_ of the text.

```c#
namespace LaubPlusCo.MarkDown.Infrastructure
{
  public class MarkDownTextService
  {
    private static readonly Regex Bold = new Regex(@"(\*\*|__) (?=\S) (.+?[*_]*) (?<=\S) \1",
      RegexOptions.IgnorePatternWhitespace | RegexOptions.Singleline | RegexOptions.Compiled);

    private static readonly Regex Italic = new Regex(@"(\*|_) (?=\S) (.+?) (?<=\S) \1",
      RegexOptions.IgnorePatternWhitespace | RegexOptions.Singleline | RegexOptions.Compiled);

    public static string Markdown(string text)
    {
      text = Bold.Replace(text, "<strong>$2</strong>");
      text = Italic.Replace(text, "<em>$2</em>");
      text = Regex.Replace(text, @"\s{2,}", "<br \\>\n");
      return text;
    }
  }
}
```

I "borrowed" the regular expressions from [MarkdownSharp](https://code.google.com/p/markdownsharp/).

Then we only need to configure which fields can contain markdown so we don't run the processor for all fields.

```xhtml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <markdownFields>
      <field>SampleHeadline</field>
      <field>SampleAbstract</field>
    </markdownFields>
    <pipelines>
      <renderField>
        <processor
          patch:after="processor[@type='Sitecore.Pipelines.RenderField.GetTextFieldValue, Sitecore.Kernel']"
          type="LaubPlusCo.MarkDown.Infrastructure.GetMarkDownTextValue, LaubPlusCo.MarkDown"/>
      </renderField>
    </pipelines>
  </sitecore>
</configuration>
```

In this example I made it extremely simple with a new configuration section which I read in code as follows. Ideally the fields should be configured in Sitecore but for now this will do.

```c#
  public class MarkDownFieldConfiguration
  {
    public static IEnumerable<string> GetMarkDownFieldNames()
    {
      var markDownFields = new List<string>();
      foreach (var configNode in Factory.GetConfigNodes("markdownFields/*").Cast<XmlNode>())
      {
        if (configNode != null && configNode.FirstChild != null && configNode.FirstChild.NodeType == XmlNodeType.Text)
          markDownFields.Add(configNode.FirstChild.Value);
      }
      return markDownFields;
    }
  }
```

## The result

Beneath you can see how markdown syntax is used in a headline and an abstract text.

[![Markdown in page edit mode](/wp-content/uploads/2014/08/markdown_in_page_edit_1.png)](/wp-content/uploads/2014/08/markdown_in_page_edit_1.png)

Now when we switch to preview, voilá..

[![Markdown in preview mode](/wp-content/uploads/2014/08/markdown_preview_1.png)](/wp-content/uploads/2014/08/markdown_preview_1.png)_Note that the Markdown syntax for a line-break is "  \\n" whilst I changed it to being just "  " so it works in a single-line field._

The code here is made as a small PoC, it worked in the first try with no testing. So in short, it works as it is but could use a bit of refactoring.

Potentially the same concept can be used with MarkdownSharp to make specific multiline text fields into page-edit friendly Markdown fields. I might make a shared source module for this, let me know if anyone would be interested in this.

That was it. My primary concern about this solution is whether or not the editors will like the syntax. I am sure they will in the case of this specific customer but I am not sure that all editors at any customer will be equally satisfied with having to learn something new.

The code can be downloaded [here](/wp-content/uploads/2014/08/LaubPlusCo.MarkDown.zip "Download code").

## Why my blog has been silent?

My loyal readers will have noticed that my blog has been almost completely silent since March this year except for one nice post written by my [colleague Alin](/author/ap/) in June.

There are many reasons for this and it is always easy to blame the innocent. Below you see a picture of my 4 month old son together with a toy hedgehog named Razl (it is definitely not their fault though). [![jonas_with_razl_the_hedgehog_1](/wp-content/uploads/2014/08/jonas_with_razl_the_hedgehog_1.png)](/wp-content/uploads/2014/08/jonas_with_razl_the_hedgehog_1.png) _Thanks to [@akamburov](https://twitter.com/akamburov) for the hedgehog. I owe you guys a review post on Razl, soon to come :)_
