---
author: alc
category:
  - sitecore
  - tips
cover:
  alt: richtext_field
  image: /wp-content/uploads/2014/10/richtext_field.jpg
date: "2014-10-23T11:22:27+00:00"
guid: http://laubplusco.net/?p=1177
title: Customizing Sitecore rich text field configuration
url: /customizing-sitecore-rich-text-field-configuration/

---
People often ask me questions on how to configure the Sitecore rich text editor in various ways.

The Sitecore rich text editor is really easy to control and customize and there are already a lot of blog posts about it out there.

Most of these posts only show how to create custom editor profiles and not how to override the actual editor configuration.

I guess everyone knows how to create custom rich text editor profiles by now so this is not what I want to show. If this is what your are looking for then just ask google.

## Overriding the default rich text editor configuration

The rich text field is a Telerik RadControl and can be controlled completely in code which is the way I prefer to do it.

This is done in Sitecore by overriding a class called Sitecore.Shell.Controls.RichTextEditor.EditorConfiguration. This class has an apply method which gets called each time a rich text editor is opened.

There are some methods on the EditorConfiguration which you would normally override to inject desired custom behaviour

### SetupFilters()

The SetupFilters method is probably the most interesting one. This method is used to set which filters to use on the content that the editors type in.

The standard SetupFilters method can enable or disable whether or not Script is allowed in rich text fields. This is controlled using the Sitecore setting called HtmlEditor.RemoveScripts

```xhtml
<!--  HTML EDITOR REMOVE SCRIPTS
	If true, the rich text editor removes script tags from RTE field values before saving. Setting the value to true reduces the potential for cross-site scripting and other script-related issues.
	Default value: true
-->
<setting name="HtmlEditor.RemoveScripts" value="true" />
```

This setting control the filter EditorFilters.RemoveScripts

It is not always sufficient to control this behaviour on all editor profiles using the setting. So it does make sense in some cases to override this method and then set the Configuration Type only on a specific Html Editor profile in core.

The below code example shows how other filters can be enabled and disabled.

```c#
protected override void SetupFilters()
{
  // Disable a filter
  Editor.DisableFilter(EditorFilters.DefaultFilters);

  // Enable filters
  Editor.EnableFilter(EditorFilters.ConvertFontToSpan);
  Editor.EnableFilter(EditorFilters.FixEnclosingP);
  Editor.EnableFilter(EditorFilters.IndentHTMLContent);
  Editor.EnableFilter(EditorFilters.MakeUrlsAbsolute);
}
```

For a complete list of EditorFilters see the Telerik documentation here [http://www.telerik.com/help/aspnet-ajax/editor-content-filters.html.](http://www.telerik.com/help/aspnet-ajax/editor-content-filters.html)

### SetupStylesheets()

The SetupStylesheets method loads in the styles to use in the rich text editor. By default Sitecore will load the stylesheet defined in the setting called WebStylesheet from configuration. This is rarely sufficient on multi site solutions.

Below is an example on how to load in more stylesheets in the rich text editor by overriding this method.

```c#
protected override void SetupStylesheets()
{
  Editor.ContentAreaCssFile = "/css/somesite/my.custom.webstylesheet.css";
  Editor.CssFiles.Add("/css/somesite/my.custom.css";);
}
```

I will not encourage you to hardcode the path as I just did in the above example but instead use Sitecore.Configuration.Settings.Get( ... ) to get your own custom configuration setting.

### SetupFontSizes()

The SetupFontSizes method is used to set what font sizes that are shown in the rich text editor drop down that the editors can choose from.

```c#
protected override void SetupFontSizes()
{
  Editor.RealFontSizes.Clear();

  for (int fontSize = 7; fontSize < 50; fontSize++)
  {
	float remSize = (float)fontSize/10;
	EditorRealFontSize editorRealFontSize = new EditorRealFontSize(string.Format(CultureInfo.InvariantCulture, "{0:F1}rem", remSize));
	Editor.RealFontSizes.Add(editorRealFontSize);
  }
}
```

For more information on Real Font sizes see Telerik's documentation here [http://www.telerik.com/help/aspnet-ajax/editor-real-font-sizes.html](http://www.telerik.com/help/aspnet-ajax/editor-real-font-sizes.html)

### SetupEditor()

The SetupEditor method is used to set a various set of general configuration options.  For example reading the newline mode from settings and setting the padding on the content area in the rich text field. It is not that common that you would like to override this method but there can be some cases where the global setting does not fit all sites.

```c#
protected override void SetupEditor()
{
  base.SetupEditor();
  // Controlled in base by setting HtmlEditor.NewLineMode
  Editor.NewLineMode = EditorNewLineModes.Div;
  Editor.Style["padding"] = "10px";
}
```

You should _ALWAYS_ call base.SetupEditor() first in your overridden method.

The EditorConfiguration class contains other methods that can be overridden as well but you would normally not have a good reason to do this.

## Configuring what EditorConfiguration type to use where

Sitecore have two levels where you can set your custom EditorConfiguration type.

You can set the general default EditorConfiguration in web.config by patching in the following Sitecore setting in an include config file.

```xhtml
<!--  HTML EDITOR DEFAULT CONFIGURATION TYPE
	Specifies the type responsible for setting up the rich text editor. Can be overriden at profile level. Must inherit from Sitecore.Shell.Controls.RichTextEditor.EditorConfiguration,Sitecore.Client.
	Default value: Sitecore.Shell.Controls.RichTextEditor.EditorConfiguration,Sitecore.Client
-->
<setting name="HtmlEditor.DefaultConfigurationType" value="Sitecore.Shell.Controls.RichTextEditor.EditorConfiguration,Sitecore.Client" />

```

Or you can set the EditorConfiguration type on the specific HtmlEditor profile as shown below.

[![editoconfiguration](/wp-content/uploads/2014/10/editoconfiguration.png)](/wp-content/uploads/2014/10/editoconfiguration.png)

This is done by creating a _Html Editor Configuration Type_ item beneath a html editor profile in core and then point the Type field to your derived class.

Which way to go depends on the solution and the desired control granularity.

## _The ToolsFile.xml and why not to use it_

You can also alter the configuration of the rich text editor by modifying the file /sitecore/shell/Controls/Rich Text Editor/ToolsFile.xml

You can see the Telerik documentation by following the link below but before you do that I will warn you about not changing this file in your Sitecore solution. This is a file that ships with Sitecore and can potentially be updated in other versions of Sitecore than the one your solution is running. The changes made in this file will also affect all rich text fields, there is no way of making it profile dependent as there is when using code.

[http://www.telerik.com/help/aspnet-ajax/editor-using-toolsfile.html](http://www.telerik.com/help/aspnet-ajax/editor-using-toolsfile.html)

That was it, I hope this post helps someone.

Please feel free to share and comment.
