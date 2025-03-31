---
author: alc
category:
  - speak
date: "2014-10-18T17:07:10+00:00"
guid: http://laubplusco.net/?p=1123
title: Setting up a Sitecore SPEAK project in Visual Studio
url: /setting-sitecore-speak-project-visual-studio/

---
Lately I've been asked a lot of questions on how to setup a project in Visual Studio for custom Sitecore SPEAK components. Here are some common issues and solutions.

## First issue - Creating the Visual Studio project with correct references

First create a .NET 4.5.x C# ASP.NET Web Application in your Visual Studio solution.

[![create_vs_project](/wp-content/uploads/2014/10/create_vs_project.png)](/wp-content/uploads/2014/10/create_vs_project.png)

In this example I place the project in a folder structure, /components/domain, beneath a Sitecore webroot.

Select the project to be empty with no references added, we will add these manually.

[![empty_web_app](/wp-content/uploads/2014/10/empty_web_app.png)](/wp-content/uploads/2014/10/empty_web_app.png)

First delete the auto-generated web.config and then add references to the following three Sitecore DLL's

- Sitecore.SPEAK.Client.dll
- Sitecore.MVC.dll
- Sitecore.Kernel.dll

Then reference all the _System_ dll's which is located in the Sitecore bin folder. By using these we are sure that we are using the same versions as Sitecore.

- System.Web.WebPages.Razor.dll
- System.Net.Http.dll
- System.Net.Http.Formatting.dll
- System.Net.Http.WebRequest.dll
- System.Web.Cors.dll
- System.Web.Helpers.dll
- System.Web.Http.Cors.dll
- System.Web.Http.dll
- System.Web.Http.WebHost.dll
- System.Web.Mvc.dll
- System.Web.Razor.dll
- System.Web.WebPages.Deployment.dll
- System.Web.WebPages.dll

Now the references are in place and you are ready to get started.

## Next issue - References does not seem to work

The .cshtml file has errors and the Model cannot be resolved. It seems like the project references are wrong.

The references are not the issue. The real problem is that you need to have a web.config that configures WebPages and razor.

You can see the web.config used by Sitecore's SPEAK applications in the folder /sitecore/shell/client/web.config it looks like this:

```xhtml
<?xml version="1.0"?>
<configuration>
  <configSections>
		<sectionGroup name="system.web.webPages.razor" type="System.Web.WebPages.Razor.Configuration.RazorWebSectionGroup, System.Web.WebPages.Razor, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35">
			<section name="host" type="System.Web.WebPages.Razor.Configuration.HostSection, System.Web.WebPages.Razor, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" requirePermission="false" />
			<section name="pages" type="System.Web.WebPages.Razor.Configuration.RazorPagesSection, System.Web.WebPages.Razor, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" requirePermission="false" />
		</sectionGroup>
  </configSections>

	<appSettings>
		<add key="webpages:Version" value="3.0.0.0" />
		<add key="ClientValidationEnabled" value="true" />
		<add key="UnobtrusiveJavaScriptEnabled" value="true" />
	</appSettings>

	<system.web.webPages.razor>
		<host factoryType="System.Web.Mvc.MvcWebRazorHostFactory, System.Web.Mvc, Version=5.1.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
		<pages pageBaseType="System.Web.Mvc.WebViewPage">
			<namespaces>
				<add namespace="System.Web.Mvc" />
				<add namespace="System.Web.Mvc.Ajax" />
				<add namespace="System.Web.Mvc.Html" />
				<add namespace="System.Web.Routing" />
				<add namespace="Sitecore.Speak" />
			</namespaces>
		</pages>
	</system.web.webPages.razor>
</configuration>

```

My solution to this issue is simply to copy paste this web.config file either directly to my project folder or to one if it's parent folders if I have several SPEAK projects within the same solution.

That solves the issue and now there are no errors in the .cshtml.

_Do not solve this issue by placing your own custom code within the /sitecore/shell/client folder. It will work but customer domain code should not go there._

## Last issue - Resolving scripts and stylesheets

The next common issue is when you have created your first component and try to see it in a browser then you'll get an error message in the console and the request for your component script returns a 404 status code.

This is because Require can't locate your script, the path is not added. To solve this you need to add the path of your new component code to the  s **peak.client.resolveScript** pipeline, see Sitecore.Speak.Config.

You add a new folder simply by patching in a new source element as shown below and then SPEAK will do it's magic.

```xhtml
<?xml version="1.0" encoding="utf-8"?>
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <pipelines>
      <speak.client.resolveScript>
        <processor type="Sitecore.Resources.Pipelines.ResolveScript.Controls, Sitecore.Speak.Client">
          <sources hint="raw:AddSource">
            <source folder="/components/domain" deep="true" category="domain" pattern="*.js,*.css" />
          </sources>
        </processor>
      </speak.client.resolveScript>
    </pipelines>
  </sitecore>
</configuration>
```

## Notes on SPEAK, Sitecore version and other resources

If you want to play around with SPEAK then use Sitecore 7.5++ where SPEAK 1.2 is introduced . This is much easier to get started with and more fun to work with. The "old" new SPEAK components and the whole business component library still works fine and you can still use the "old" approaches which you might have seen in some of my earlier posts.

SPEAK 1.2 also introduce TypeScript typings for SPEAK and a lot of other goodies. I have a blog post coming up tomorrow on these goodies and another one soon on [the Bootstrap SPEAK component library by Jakob Christensen](https://github.com/JakobChristensen/Sitecore.Speak.Bootstrap3.SDK). These posts will show some of the advantages that the new version 1.2 of SPEAK offers.

You should also read my old posts that [introduces SPEAK components](/introducing-sitecore-speak-components-some-basics/ "Part 1: Introducing Sitecore SPEAK Components") , _note that they all use the "old" SPEAK 1.1._

For a bit more in-depth details into how SPEAK works behind the scenes then read my post on [Sitecore SPEAK rules](/sitecore-speak-rules-magic/ "Sitecore SPEAK rules magic") magic and my other post on [Sitecore SPEAK pipelines](/making-sense-speak-pipelines/ "Making sense of Sitecore SPEAK pipelines"). The pipeline concept I sketched in that post is now a part of Sitecore 8 in a slightly modified version.

If you need a really good 1-0-1 in how to [create SPEAK applications then read Martina Welanders great series of posts](http://mhwelander.net/category/speak/).

Also take a look at [Pierre Durval's best practices in Sitecore SPEAK](https://github.com/Sitecore-Community/Sitecore.Speak.Guideline/blob/master/bestPractises.md), these are really a good read for us C# developers.

Jakob Christensen also made a ton of [great Sitecore SPEAK video material which he posted on youtube](https://www.youtube.com/user/JakobHChristensen).

I hope this post will help more developers get started on SPEAK. Drop me a comment if there is something you'll like me to add or other comments to this post.

Also please remember to share ;)
