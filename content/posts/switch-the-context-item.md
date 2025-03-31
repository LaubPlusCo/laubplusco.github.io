---
author: alc
category:
  - sitecore
  - sitecore-basics
  - tips
date: "2013-11-07T22:19:09+00:00"
guid: http://laubplusco.net/?p=284
title: Switch the Context Item
url: /switch-context-item/

---
Some times you'll come across some code which relies on reading the Sitecore.Context.Item and there is no way for you to change this.

This can occur when calling some standard Sitecore pipelines, some custom code which has been inherited without the source etc.

Well, Sitecore API to the rescue.

## Introducing the ContextItemSwitcher

I am not sure how long this has been a part of the Sitecore API, but at least since 6.4 where I first noticed it.

Lets say that you want to run the insertRenderings pipeline for another item than the context item (for whatever reason). This code is taken a bit out-of-context in many ways but it serves well as an example.

```c#
public IList<RenderingReference> ReadRenderingsFromItem(Item item)
{
  var pipelineArgs = new InsertRenderingsArgs();
  using (new ContextItemSwitcher(item))
  {
    CorePipeline.Run("insertRenderings", pipelineArgs);
  }
  return pipelineArgs.Renderings;
}
```

The first thing the insertRenderings pipeline does in the processo _r Sitecore.Pipelines.InsertRenderings.Processors.GetItem_ is to set the InsertRenderingsArgs.Item to the Context.Item.

Using a ContextItemSwitcher when starting the pipeline makes it possible to run the pipeline out of context without having to modify the individual processors. Quite smart,  some might even call it a hidden gem in the Sitecore Api.
