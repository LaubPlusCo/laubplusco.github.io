---
author: alc
category:
  - sitecore
  - sitecore-8
  - sitecore-mvc
  - tips
date: "2015-09-24T13:29:05+00:00"
guid: http://laubplusco.net/?p=1515
title: How to extend placeholders in Sitecore MVC
url: /how-to-extend-placeholders-in-sitecore-mvc/

---
_Wow it has been ages since I last wrote a blog post so it is about time that I get started again._

A colleague of mine asked me a question the other day about how I would implement a page(experience)-editable accordion spot where each element in the accordion is a rendering item. Kind of like shown below where the content in each accordion element is a rendering with its own datasource. You should be able to personalize and test each rendering individually.

![accordion_example](/wp-content/uploads/2015/09/accordion_example.png)

There is a lot of solutions to achieve this but to keep it close to the Sitecore API and to make it work in the Experience Editor I thought how about implementing an accordion placeholder? That is a placeholder that automatically wraps each rendering within it, in <li> elements within an outer <ul> element.

Back in the webforms days this would have been tricky to do but now with the MVC API it is really easy. No control tree that has to be built up, just a glorified text writer.

First we reflect our way into the placeholder extension method that renders out a normal Sitecore placeholder.

```c#
    public virtual HtmlString Placeholder(string placeholderName)
    {
      Assert.ArgumentNotNull((object) placeholderName, "placeholderName");
      using (ContextService.Get().Push<ViewContext>(this.HtmlHelper.ViewContext))
      {
        StringWriter stringWriter = new StringWriter();
        PipelineService.Get().RunPipeline<RenderPlaceholderArgs>("mvc.renderPlaceholder", new RenderPlaceholderArgs(placeholderName, (TextWriter) stringWriter, this.CurrentRendering));
        return new HtmlString(stringWriter.ToString());
      }
    }
```

This code simply calls the mvc.renderPlaceholder pipeline. Then if we take a closer look at the last processor in this pipeline called Sitecore.Mvc.Pipelines.Response.RenderPlaceholder.PerformRendering

```c#
namespace Sitecore.Mvc.Pipelines.Response.RenderPlaceholder
{
  public class PerformRendering : RenderPlaceholderProcessor
  {
    public override void Process(RenderPlaceholderArgs args)
    {
      Assert.ArgumentNotNull((object) args, "args");
      this.Render(args.PlaceholderName, args.Writer, args);
    }

    protected virtual void Render(string placeholderName, TextWriter writer, RenderPlaceholderArgs args)
    {
      foreach (Rendering rendering in this.GetRenderings(placeholderName, args))
        PipelineService.Get().RunPipeline<RenderRenderingArgs>("mvc.renderRendering", new RenderRenderingArgs(rendering, writer));
    }

    ....
  }
}
```

As you can see this processor simply iterates over the renderings placed in the placeholder and render them by calling the renderRendering pipeline.

The really cheap and dirty solution would be simply to override this processor and given some condition (placeholder name starts with accordion) throw in a <ul> and <li> tags around each rendering and job's done.

```c#
  public class PerformAccordionRendering : PerformRendering
  {
    private const string AccordionPlaceholderKey = "accordion";

    protected virtual void Render(string placeholderName, TextWriter writer, RenderPlaceholderArgs args)
    {
      if (!placeholderName.StartsWith(AccordionPlaceholderKey))
      {
        base.Render(placeholderName, writer, args);
        return;
      }

      writer.Write("<ul>");
      foreach (Rendering rendering in this.GetRenderings(placeholderName, args))
      {
        writer.Write("<li>");
        PipelineService.Get().RunPipeline("mvc.renderRendering", new RenderRenderingArgs(rendering, writer));
        writer.Write("</li>");
      }
      writer.Write("</ul>");
    }
  }
```

I prefer pragmatism and clean code over cheap and dirty so I would like to take the concept a bit further just for fun.

## Making it prettier

The RenderPlaceholderArgs inherit from good old PipelineArgs that has a customdata dictionary so we have a way of passing data to the pipeline.

First we create a simple interface called IRenderPlaceholder that has a method Render.

```c#
  public interface IRenderPlaceHolder
  {
    void Render(TextWriter writer, IEnumerable<Rendering> renderings);
  }
```

Then we override the PerformRendering processor and implement a check if the custom data dictionary contains a key for a specific placeholder renderer and then we use [ReflectionUtil](/simple-ioc-container-using-only-the-sitecore-api/) to look up this type and try to cast it to our IRenderPlaceholder interface.

```c#
  public class PerformRendering : Sitecore.Mvc.Pipelines.Response.RenderPlaceholder.PerformRendering
  {
    public const string RenderPlaceholderTypeKey = "rt";
    protected override void Render(string placeholderName, TextWriter writer, RenderPlaceholderArgs args)
    {
      if (!args.CustomData.ContainsKey(RenderPlaceholderTypeKey))
      {
        base.Render(placeholderName, writer, args);
        return;
      }
      var renderer = (IRenderPlaceHolder)ReflectionUtil.CreateObject((Type)args.CustomData[RenderPlaceholderTypeKey]);
      Assert.IsNotNull(renderer, "Could not instantiate custom placeholder renderer for placeholder " + placeholderName);
      renderer.Render(writer, GetRenderings(placeholderName, args));
    }
  }
```

Finally we patch our new processor into the renderPlaceholder pipeline instead of Sitecore.Mvc.Pipelines.Response.RenderPlaceholder.PerformRendering

```xhtml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <pipelines>
      <mvc.renderPlaceholder >
        <processor patch:instead="processor[@type='Sitecore.Mvc.Pipelines.Response.RenderPlaceholder.PerformRendering, Sitecore.Mvc']"
                   type="[NAMESPACE].PerformRendering, [ASSEMBLY]" />
      </mvc.renderPlaceholder>
    </pipelines>
  </sitecore>
</configuration>
```

 _**Note**: Ensure that this patch is read in last. Either by placing the include config file in a folder beneath /App\_Config/Include or by prefixing the filename with zzz._

Next up we create our AccordionPlaceholderRenderer that implements our IRenderPlaceholder:

```c#
  public class RenderAccordionPlaceholder : IRenderPlaceHolder
  {
    public void Render(TextWriter writer, IEnumerable<Rendering> renderings)
    {
      writer.Write("<ul>");
      foreach (Rendering rendering in renderings)
      {
        writer.Write("<li>");
        PipelineService.Get().RunPipeline<RenderRenderingArgs>("mvc.renderRendering", new RenderRenderingArgs(rendering, writer));
        writer.Write("</li>");
      }
      writer.Write("</ul>");
    }
  }
```

Then we write a new html helper extension method that renders out an accordion placeholder by adding the typename to the custom data dictionary.

```c#
  public static class AccordionPlaceholderExtensions
  {
    public static HtmlString AccordionPlaceholder(this HtmlHelper helper, string placeholderName)
    {
      Assert.ArgumentNotNull(placeholderName, "placeholderName");
      using (ContextService.Get().Push<ViewContext>(helper.ViewContext))
      {
        var stringWriter = new StringWriter();
        var args = new RenderPlaceholderArgs(placeholderName, stringWriter, helper.Sitecore().CurrentRendering);
        args.CustomData.Add(PerformRendering.RenderPlaceholderTypeKey, typeof(RenderAccordionPlaceholder));
        PipelineService.Get().RunPipeline("mvc.renderPlaceholder", args);
        return new HtmlString(stringWriter.ToString());
      }
    }
  }
```

And voila, all rendering items that are placed within this placeholder now renders out in a list :)

![accordion_spot](/wp-content/uploads/2015/09/accordion_spot.png)

_**Tips**_, I've received some feedback with some handy tips for the accordion placeholder.

1. Turn off the accordion Javascript in page edit mode so it is always fully expanded. We typically put a class on body indicating if the page is in page edit mode.
1. When in page edit mode add some margin to the ul and li elements so these can be clicked in the experience editor. Otherwise you'll need to navigate up the hierarchy by clicking an inner element and using the navigation on the floating toolbar

## _A little bit of bonus.._

Now you can also easily make an extended placeholder that wraps all it's rendering items in let's say  a <section> element:

```c#
  public class RenderSectionPlaceholder : IRenderPlaceHolder
  {
    public void Render(TextWriter writer, IEnumerable<Rendering> renderings)
    {
      foreach (Rendering rendering in renderings)
      {
        writer.Write("<section>");
        PipelineService.Get().RunPipeline<RenderRenderingArgs>("mvc.renderRendering", new RenderRenderingArgs(rendering, writer));
        writer.Write("</section>");
      }
    }
  }
```

Or a placeholder that only render out 3 renderings at random when not in page edit mode:

```c#
  public class RenderThreeRandomSpotsPlaceholder : IRenderPlaceHolder
  {
    public void Render(TextWriter writer, IEnumerable<Rendering> renderings)
    {
      if (Sitecore.Context.PageMode.IsPageEditor)
        RenderAllRenderings(writer, renderings);
      else
        RenderThreeRandomRenderings(writer, renderings.ToList());
    }

    private void RenderThreeRandomRenderings(TextWriter writer, IList<Rendering> renderings)
    {
      if (renderings.Count <= 3)
        RenderAllRenderings(writer, renderings);
      var randomizer = new Random();
      for (var i = 0; i < 3; i++)
      {
        var index = randomizer.Next(renderings.Count);
        PipelineService.Get().RunPipeline("mvc.renderRendering", new RenderRenderingArgs(renderings[index], writer));
        renderings.RemoveAt(index);
      }
    }

    private void RenderAllRenderings(TextWriter writer, IEnumerable<Rendering> renderings)
    {
      foreach (Rendering rendering in renderings)
      {
        PipelineService.Get().RunPipeline<RenderRenderingArgs>("mvc.renderRendering", new RenderRenderingArgs(rendering, writer));
      }
    }
  }
```

And so on and so forth. Just make a new HtmlHelper extension method for each type or implement a generic method where the type is IRenderPlaceholder.

That was it, I hope to blog some more the coming weeks, I've been missing it.
