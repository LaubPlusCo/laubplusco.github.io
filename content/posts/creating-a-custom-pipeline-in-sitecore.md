---
author: alc
category:
  - pipelines
  - sitecore
  - sitecore-basics
  - tips
date: "2013-10-25T14:04:56+00:00"
guid: http://laubplusco.net/?p=223
title: Creating a custom pipeline in Sitecore
url: /creating-custom-pipeline-in-sitecore/

---
Pipelines are one of the most essential parts of Sitecore and creating your own custom pipeline in Sitecore makes your code extremely flexible for both you and others. It is extremely easy to create and run a custom pipeline as this post will show.

## Defining the pipeline

A pipeline consist is a set of processor classes which each has a method called Process which takes one argument of PipelineArgs or a derived class.

To configure a pipeline create a new .config file in the include folder:

```xhtml
<configuration>
  <sitecore>
    <pipelines>
      <somePipeline>

      </somePipeline>
    </pipelines>
  </sitecore>
</configuration>
```

Create a class derived from PipelineArgs which suits your implementation needs.

```c#
  public class SomePipelineArgs : PipelineArgs
  {
    public string Result { get; set; }
  }
```

Create the processor class(es):

```c#
public class SomeProcessor
{
  public void Process(SomePipelineArgs args)
  {
    args.Result = GetResult();
  }

  private string GetResult()
  {
    return "Hellow world";
  }
}
```

Insert them in your config.

```xhtml
      <somePipeline>
        <processor type="[NAMESPACE].SomeProcessor, [ASSEMBLY]" />
      </somePipeline>
```

## Calling the pipeline

To call the pipeline we use the CorePipeline.Run method as follows:

```c#
    var pipelineArgs = new SomePipelineArgs();
    CorePipeline.Run("somePipeline", pipelineArgs);
    Log.Info(args.Result, this);
```

This is how simple it is to create a pipeline.
