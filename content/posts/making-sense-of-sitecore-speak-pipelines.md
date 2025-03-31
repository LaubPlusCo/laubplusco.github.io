---
author: alc
category:
  - pipelines
  - sitecore-7.1
  - speak
date: "2014-01-30T23:42:40+00:00"
guid: http://laubplusco.net/?p=672
title: Making sense of Sitecore SPEAK pipelines
url: /making-sense-speak-pipelines/

---
Following this [great post](http://blog.coates.dk/2014/01/22/speak-pipelines/) by my colleague Alan I came up with an idea on how to improve the SPEAK pipelines concept.

The main problem with the pipelines as they are in Sitecore 7.1 is that there is no way to add or remove processors without making changes in JavaScript code. You can't even change the order in which they are added without changing their priority in code (see Alan's [blog post](http://blog.coates.dk/2014/01/22/speak-pipelines/)).

_**Update**: This concept is now a part of the new SPEAK version with small updates. It will be available in Sitecore 8. Thanks Sitecore for reading my blog. More on this implementation in a soon-to-come post._

## Configuring pipelines

First I've added two templates, one for a SPEAK-Pipeline and one for a SPEAK-Pipeline-processor.

The SPEAK pipeline template contains one field for the name of the pipeline, I don't like using the item name for this and a regex ensuring no spaces etc. should be added. The pipeline item's standard values has insert options set for SPEAK-Pipeline-Processor items.

The SPEAK-Pipeline-processor template have two fields, one for the path to a processor JS file and one for the name of the processor (the object name used in the JS file, will be removed later).

[![Speak Pipeline templates](/wp-content/uploads/2014/01/SpeakPipelineTemplates1.png)](/wp-content/uploads/2014/01/SpeakPipelineTemplates1.png)

Now pipelines can be configured, not in config, but in Sitecore. The concept is then to add a Pipelines folder beneath the app in which the pipeline is used.

This approach sticks to the concept of keeping everything confined within an application to get rid of potential dependency spaghetti.

[![Pipeline Items](/wp-content/uploads/2014/01/PipelineItems.png)](/wp-content/uploads/2014/01/PipelineItems.png)

## The pipeline component

Then I've created a new SPEAK component called Pipeline. This component only renders out a div tag with a require attribute that points to /-/speak/v1/pipelines/pipeline.js?id=<the id of the rendering datasource>

The component also renders out three optional rendering parameters which I bind to the model. One called pipelineArgs, one called trigger and one called targetControl.

```c#
@using Sitecore.Configuration
@using Sitecore.Data
@using Sitecore.Diagnostics
@using Sitecore.Mvc
@using Sitecore.Mvc.Presentation
@using Sitecore.Text
@using Sitecore.Web.UI.Controls.Common.UserControls
@model RenderingModel
@{
    var rendering = Html.Sitecore().Controls().GetUserControl(Model.Rendering);
    Assert.IsNotNullOrEmpty(rendering.DataSource, "Datasource is missing on pipeline rendering " + rendering.ControlId);
    rendering.Class = "sc-Pipeline";
    rendering.Requires.Script("controls", "Pipeline.js");

    var pipelineScriptUrl = new UrlString(SpeakSettings.Html.RequireJsCustomHandler + "pipelines/pipeline.js");
    pipelineScriptUrl.Add("id", rendering.DataSource);
    rendering.Requires.Script(pipelineScriptUrl.ToString());

    var pipelineItem = Factory.GetDatabase("core").GetItem(new ID(rendering.DataSource));
    var pipelineName = pipelineItem["PipelineName"];
    if (string.IsNullOrEmpty(pipelineName))
    {
        pipelineName = pipelineItem.Name;
    }
    rendering.SetAttribute("data-sc-pipelinename", pipelineName);

    rendering.SetAttribute("data-sc-pipelineargs", rendering.GetString("PipelineArgs", "pipelineArgs", string.Empty));
    rendering.SetAttribute("data-sc-trigger", rendering.GetString("Trigger", "trigger", string.Empty));
    rendering.SetAttribute("data-sc-targetcontrol", rendering.GetString("TargetControl", "targetControl", string.Empty));
    rendering.DataBind = "pipelineArgs: pipelineArgs";
    var htmlAttributes = rendering.HtmlAttributes;
}
<div @htmlAttributes>
</div>
```

The pipelineArgs can be used to bind the arguments which is passed to the pipeline with another model attribute. The trigger can be used to trigger the pipeline if the event written as trigger is triggered on the targetControl.

```js
define(["sitecore"], function(Sitecore) {
  Sitecore.Factories.createBaseComponent({
    name: "Pipeline",
    base: "ComponentBase",
    selector: ".sc-Pipeline",
    attributes: [
      { name: "pipelineName", value: "$el.data:sc-pipelinename" },
      { name: "pipelineArgs", value: "$el.data:sc-pipelineargs" },
      { name: "trigger", value: "$el.data:sc-trigger" },
      { name: "targetControl", value: "$el.data:sc-targetcontrol" }
    ],
    initialize: function () {
      this._super();
      var trigger = this.model.get("trigger");
      if (trigger === null || trigger === "")
        return;
      var targetControl = this.getTargetControl();
      console.log(targetControl);
      if (targetControl !== undefined) {
        targetControl.on(trigger, this.startPipeline, this);
      }
      this.app.on(trigger, this.startPipeline, this);
    },
    startPipeline: function () {
      var context = {};
      context.args = this.model.get("pipelineArgs");
      context.app = this.app;
      context.pipeline = this;
      Sitecore.Pipelines[this.model.get("pipelineName")].execute(context);
    },
    getTargetControl: function() {
      var targetControlName = this.model.get("targetControl");
      if (targetControlName === null || targetControlName === "")
        return;
      return this.app[targetControlName];
    }
  });
});
```

I really like this binding concept. UI events can trigger a client side pipeline making them potentially extremely powerful and still extremely easy to inject your own custom code into.

## Creating the dynamic pipelines

So last thing to get this concept up and running is to dynamically generate the Javascript which adds the processors to the pipeline.

I got inspired by the [rules javascript magic](/sitecore-speak-rules-magic/ "Sitecore SPEAK rules magic") and created a new server side processor for the resolveScript pipeline defined in Sitecore.Speak.config.

```c#
namespace LaubPlusCo.ConfiguredPipelines
{
  public class PipelineScriptProcessor : ResolveScriptProcessor
  {
    private static readonly ID ProcessorFilePathFieldId = new ID("{20D0BB88-0649-4714-A6DD-585DD4CA5B4E}");
    private static readonly ID PipelineNameFieldId = new ID("{F0F36FDD-D818-4A1B-8273-7B1D1E7EE053}");
    private static readonly ID ProcessorNameFieldId = new ID("{9E91BAA8-5CAC-48E0-8AB2-04219118BF08}");

    public override void Process(ResolveScriptArgs args)
    {
      Assert.ArgumentNotNull(args, "args");
      if (!args.FileName.StartsWith("pipelines/pipeline.js", StringComparison.InvariantCultureIgnoreCase))
        return;
      args.AbortPipeline();
      var urlString = new UrlString(args.FileName);
      var pipelineId = HttpUtility.UrlDecode(urlString["id"]);
      if (string.IsNullOrEmpty(pipelineId) || !ID.IsID(pipelineId))
        return;

      var pipelineItem = Factory.GetDatabase("core").GetItem(pipelineId);

      // Note: should check if item IsDerived from pipeline
      var processors = GetProcessors(pipelineItem);
      if (!processors.Any())
        return;

      var script = GetDefineScript(processors);
      if (string.IsNullOrEmpty(script))
        return;
      var pipelineName = pipelineItem[PipelineNameFieldId];
      if (string.IsNullOrEmpty(pipelineName))
        pipelineName = pipelineItem.Name;
      script = AddPipelineProcessors(pipelineName, processors, script);

      var bytes = Encoding.UTF8.GetBytes(script.ToCharArray());
      args.Content = new MemoryStream(bytes);
    }

    private string AddPipelineProcessors(string pipelineName, IList<Item> processors, string script)
    {
      var writer = new StringWriter(new StringBuilder(script));
      writer.WriteLine(", function(Sitecore) {");

      writer.WriteLine("  Sitecore.Pipelines." + pipelineName + " = Sitecore.Pipelines." + pipelineName + " || new Sitecore.Pipelines.Pipeline(\"" + pipelineName + "\");");
      for (var i = 0; i <processors.Count; i++)
      {
        var processorName = processors[i][ProcessorNameFieldId];
        if (string.IsNullOrEmpty(processorName))
          processorName = processors[i].Name;
        writer.WriteLine("  " + processorName + ".priority=" + (i + 1) + ";");
        writer.WriteLine("  " + processorName + ".name=\"" + processorName + "\";");
        writer.WriteLine("  Sitecore.Pipelines." + pipelineName + ".add(" + processorName + ");");
      }

      writer.WriteLine("});");

      return writer.ToString();
    }

    private string GetDefineScript(IEnumerable<Item> processors)
    {
      var processorPaths = processors.Select(p => "\"" + p[ProcessorFilePathFieldId] + "\"").ToArray();
      return "define([\"sitecore\"," + string.Join(",", processorPaths) + "]";
    }

    private List<Item> GetProcessors(Item pipelineItem)
    {
      var processors = new List<Item>();

      // Note: should check if child item IsDerived from processor
      foreach (var processor in pipelineItem.GetChildren().Select(child => (Item) child))
      {
        var processorScriptPath = processor[ProcessorFilePathFieldId];
        if (!string.IsNullOrEmpty(processorScriptPath) && FileUtil.Exists(processorScriptPath))
          processors.Add(processor);
      }
      return processors;
    }
  }
}
```

And here is the outputted JavaScript, really simple even though the C# code could need a real good cleaning (not now)..

```js
define(["sitecore","/Components/ConfiguredPipelines/Processors/ProcessorOne.js","/Components/ConfiguredPipelines/Processors/ProcessorTwo.js"], function(Sitecore) {
  Sitecore.Pipelines.TestPipeline = Sitecore.Pipelines.TestPipeline || new Sitecore.Pipelines.Pipeline("TestPipeline");
  processorOne.priority=1;
  processorOne.name="processorOne";
  Sitecore.Pipelines.TestPipeline.add(processorOne);
  processorTwo.priority=2;
  processorTwo.name="processorTwo";
  Sitecore.Pipelines.TestPipeline.add(processorTwo);
});
```

I've added the processor in the Sitecore.Speak.Config file just after the rule processor which as mentioned has been a source for inspiration. This should of course be patched in a separate config include file if this code is used by someone other than Sitecore.

```xhtml
      <speak.client.resolveScript>
        <processor type="Sitecore.Resources.Pipelines.ResolveScript.Main, Sitecore.Speak.Client" />
        <processor type="Sitecore.Resources.Pipelines.ResolveScript.Rule, Sitecore.Speak.Client" />
        <processor type="LaubPlusCo.ConfiguredPipelines.PipelineScriptProcessor, LaubPlusCo.ConfiguredPipelines" />
        <processor type="Sitecore.Resources.Pipelines.ResolveScript.ResolveBaseComponent, Sitecore.Speak.Client" />
        <processor type="Sitecore.Resources.Pipelines.ResolveScript.Controls, Sitecore.Speak.Client">
```

As you saw in the pipeline component implementation we then let require do it's magic by requiring the outputted JavaScript making sure that we have the pipeline and all the processors at our disposal.

The pipeline can now be run either by binding it to some pipelineArgs object and setting a trigger on the rendering parameters or it can be started in the PageCode or other places like this:

```js
Sitecore.Pipelines.<pipeline name>.execute(context);
```

## How is a processor written then?

```js
var processorOne =
{
  execute: function(context) {
    // Processor code
  }
};
```

There is one small issue with this implementation which I will fix later by implementing another resolveScript processor.

The issue is that the processor object has to be named exactly like it is written on the processor item in the processor name field. Otherwise the JavaScript will fail as it is implemented now. It is a bit redundant but the fix is clear.

## Finally

Sitecore, please use this concept (maybe for a good bottle of red wine or some shares :-) ). The code will need some polishing though, but the whole binding client events to trigger pipelines could be very powerful and make it extremely easy to inject custom code into the UI by adding new processors in Sitecore.

One might also argue that the same functionality could be achieved by making a rule action which executed a pipeline (I can feel yet another a blog post coming up soon). The CallFunction action from this [post](/creating-custom-sitecore-speak-rule-conditions-actions/ "Creating custom Sitecore SPEAK rule Conditions and Actions") could actually also be used together with the pipeline rendering to call the function startPipeline and there you have it already, executing pipelines from a rule.

If the concept is implemented in Sitecore then it could also be used for "Core" SPEAK pipelines as well which then would be shared between app's for common functionality. Just be aware that they really need to be real system pipelines to reduce inflexibility of the API in the future.

In a follow up post I will show an example usage of this pipeline implementation and who knows maybe also make sure that the processors can be dynamically named from the items so we get rid of the name redundancy.
