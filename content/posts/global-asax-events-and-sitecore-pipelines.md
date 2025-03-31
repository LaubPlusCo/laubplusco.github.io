---
author: alc
category:
  - best-practices
  - pipelines
  - sitecore
cover:
  alt: initpipeline
  image: /wp-content/uploads/2014/10/initpipeline.png
date: "2014-10-12T17:33:49+00:00"
guid: http://laubplusco.net/?p=1110
title: Global.asax events and Sitecore pipelines
url: /global-asax-sitecore-pipelines/

---
At Pentia we always try not to change any files that ships in an out-of-the-box Sitecore installation. This makes upgrading easier and the solution code more transparent for developers.

Sitecore ships with a standard Global.asax file and a lot of Sitecore developers choose to make changes directly in this to hook into application and session level events.

For example see [the tutorial code from Glass Sitecore Mapper](https://github.com/Glass-lu/Glass.Sitecore.Mapper/blob/master/Tutorials/Glass.Sitecore.Mapper.Tutorial/Global.asax.cs) here the Global.asax is used to set the glass context on the Application\_Start event.

This could be done in the Sitecore **initialize** pipeline instead of changing the Global.asax file. _It would actually be nice to see the Glass example code moved to the initialize pipeline instead so it fully embraces the Sitecore API :)_

I recently worked on a solution Pentia inherited from another Sitecore partner. This solution used the Global.asax to initialize IoC containers through some class on the Application\_Start event as shown below.

```c#
  public void Application_Start()
  {
    new Initializer().Init();
  }
```

The first thing I did when I saw that code was to move it to where I personally feel it belongs that is in a processor patched into the initialize pipeline. I still dislike their class name, _Initializer_, but that is something else.

```c#
  public class InitializeInversionOfControlContainers
  {
    public void Process(PipelineArgs args)
    {
      new Initializer().Init();
    }
  }

```

Sitecore starts the **initialize** pipeline on Application\_Start so there is nothing that you can do on the Global.asax Application\_Start event that you cannot do in a processor in the initialize pipeline.

The same goes for the Application\_End event and the **shutdown** pipeline. Anything you would like to do on Application\_End can go in a processor in the shutdown pipeline.

The list below shows which Sitecore pipelines that can be used instead of Global.asax event handlers.

- Application\_Start <> **initialize**
- Application\_BeginRequest <> **preprocessRequest** / **httpRequestBegin**
- Application\_EndRequest <> **httpRequestEnd**
- Session\_End <> **sessionEnd**
- Application\_End <> **shutdown**

There is no pipeline for Application\_Error but Sitecore logs these anyway so I very rarely see a valid reason for hooking into this event.

## Why use the standard Sitecore pipelines?

Why not use Global.asax when we all know that it works fine.

It is of course a bit subjective but I prefer to fully embrace the Sitecore API and keep responsibility in one place only. So everything that happens when the application starts can be seen in the initialize pipeline, everything that happens when the application ends can be seen in the shutdown pipeline and so on. Please feel free to share your thoughts on this in the comments.
