---
author: ap
category:
  - sitecore
date: "2014-10-07T06:57:46+00:00"
guid: http://laubplusco.net/?p=1064
title: Insert Sitecore related content directly into the RTE
url: /insert-sitecore-related-content-directly-rte/

---
What I want to achieve with this post is give the editors the possibility of inserting related content from other pages into their current Rich Text Editor. They could insert related content from some selected fields in another pages or render some spots so that they would present a page somewhat like this : [![demospot](/wp-content/uploads/2014/09/demospot-300x176.jpg)](/wp-content/uploads/2014/09/demospot.jpg)

I am still working on mastering the art of explaining in text so  I will try to make it as short and concise as possible. Green text comes from a _”Page 1”_ and gray text is related content inserted from _”Page 2”_ . So on _"Page 1"_ we will not have to have a RTE field, then a related content selector and another RTE field. There will be just one RTE field.

The solution is to insert a custom snippet in the RTE which then I will capture in the RenderField Pipeline and replace it with the markup I want.

The snippet I created is in the Core db and has this value:

[![value](/wp-content/uploads/2014/09/value-300x60.jpg)](/wp-content/uploads/2014/09/value.jpg)

This means that the editor will need to know the Page Item ID from where the gray text will be rendered from. I selected this XML formatting for ease of explanation but it can be anything you can later identify and extract from the text.

Then I created my own Pipeline step (RenderSublayout) and inserted it right after the ” _GetTextFieldValue”_ pipeline step in the RenderField pipeline.

```c#
<renderField>
        <processor type="Sitecore.Pipelines.RenderField.SetParameters, Sitecore.Kernel"/>
        <processor type="Sitecore.Pipelines.RenderField.GetFieldValue, Sitecore.Kernel"/>
        <processor type="Sitecore.Pipelines.RenderField.GetTextFieldValue, Sitecore.Kernel"/>
        <processor type="SpotInRichTextEditor.Infrastructure.RenderSublayout, SpotInRichTextEditor"/>
        <processor type="Sitecore.Pipelines.RenderField.ExpandLinks, Sitecore.Kernel"/>
...
</renderField>
```

In this class I am finding all the snippets  that I want to replace with my own markup and assemble them in a list of _RelatedContentSpotSnippets_. This way the editor can add more of these snippets in the same RTE and they will be replaced with the correct markup.

```c#
public class RenderSublayout
 {
 public void Process(RenderFieldArgs args)
 {
 if (args.FieldTypeKey != "rich text" || string.IsNullOrEmpty(args.FieldValue))
 return;

 if (!ContentShouldRenderSpot(args.FieldValue))
 return;

 AssembleSpotIdList(args);
 }

 private bool ContentShouldRenderSpot(string fieldValue)
 {
 return fieldValue.ToLower().Contains("your_page_id_here");
 }

 private void AssembleSpotIdList(RenderFieldArgs args)
 {
 List<RelatedContentSpotSnippet> spotIds = new List<RelatedContentSpotSnippet>();
 var matchCollection = Regex.Matches(args.FieldValue, @"<your_page_id_here[^>]*>(.*?)<\/your_page_id_here>");
 foreach (Match match in matchCollection)
 {
 string relatedContentSpotId = Regex.Matches(match.ToString(), @"{(.*?)}")[0].ToString();
 ID newId;
 if (string.IsNullOrEmpty(relatedContentSpotId)) continue;
 if (ID.TryParse(relatedContentSpotId, out newId))
 spotIds.Add(new RelatedContentSpotSnippet
 {
 Match = match,
 Position = args.FieldValue.IndexOf(match.Value, StringComparison.Ordinal),
 SpotId = newId
 });
 }

 if (spotIds.Any())
 RenderSpots(args, spotIds);
 }

 private void RenderSpots(RenderFieldArgs args, IEnumerable<RelatedContentSpotSnippet> spotSnippets)
 {
 using (new SecurityDisabler())
 {
 foreach (RelatedContentSpotSnippet spotSnippet in spotSnippets)
 {
 Item item = Context.ContentDatabase.GetItem(spotSnippet.SpotId);
 string userControlRendering =
 RenderSublayoutService.RenderUserControl<Spot>(
 "/Components/SpotInRichTextEditor/Presentation/Spot.ascx", item);

 args.Result.FirstPart = args.FieldValue.Replace(spotSnippet.Match.ToString(), userControlRendering);
 }
 }
 }
 }

 internal class RelatedContentSpotSnippet
 {
 public int Position { get; set; }
 public ID SpotId { get; set; }
 public Match Match { get; set; }
 }

```

This approach can be modified so that instead of inserting the ItemId the editor would insert the path to the spot. Also one may insert an image representing the spot so that the editor has an idea on how the page woud look.

The _RenderUserControlService_ looks like this:

```c#
 public class RenderSublayoutService
  {
    public static string RenderSublayout<T>(string controlPath, Item dataSource) where T : UserControl, IDataSourceItem
    {
      var pageHolder = new Page();
      var controlToRender = (T)(pageHolder.LoadControl(controlPath));
      controlToRender.DataSource = dataSource;
      pageHolder.Controls.Add(controlToRender);
      var result = new StringWriter();
      HttpContext.Current.Server.Execute(pageHolder, result, false);
      return result.ToString();
    }
  }

  public interface IDataSourceItem
  {
    Item DataSource { get; set; }
  }
```

I think this functionality can be extended depending on the requirements but at least I consider it to be a start for someone who wants to achieve something like this.
