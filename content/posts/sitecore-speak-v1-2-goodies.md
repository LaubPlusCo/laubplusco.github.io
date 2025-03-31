---
author: alc
category:
  - sitecore
  - speak
cover:
  alt: speak_goodies
  image: /wp-content/uploads/2014/10/speak_goodies.png
date: "2014-10-19T15:32:12+00:00"
guid: http://laubplusco.net/?p=1136
title: Sitecore SPEAK v1.2 goodies
url: /sitecore-speak-v1-2-goodies/

---
This post show some of the new goodies we Sitecore developers get when working with Sitecore SPEAK 1.2 and Sitecore Rocks compared with the previous version of SPEAK.

## Strongly typed View Model

In SPEAK 1.1 there was a lot of plumbing work you had to do almost all over the place.

In your cshtml view you first had to get the user control from the Sitecore Mvc RenderingModel. Then you could start building up your view by getting the rendering parameters from the model by calling various Get methods with loosely typed parameters.

The below code shows an example from one of previous posts.

```c#
@model RenderingModel
@{
    var userControl = Html.Sitecore().Controls().GetUserControl(Model.Rendering);
    userControl.Requires.Script("speakexamples", "ExampleControl.js");
    userControl.Requires.Css("speakexamples", "ExampleControl.css");
    userControl.Class = "example-control";
    var displayedText = userControl.GetString("DisplayedText", "displayedText", "Some text");
    userControl.Attributes.Add("data-sc-displayedtext", displayedText);
    var htmlAttributes = userControl.HtmlAttributes;
}
<div @htmlAttributes>
    <div class="example-text" data-bind="text: displayedText">@displayedText</div>
    <div class="example-count" data-bind="text: count">0</div>
</div>
```

It was also necessary to explicitly set what scripts and stylesheet the view required even though the SPEAK guidelines made you place the files in the same folder with the same name as the cshtml file.

Now in 1.2 your model class is derived from a base class called Sitecore.Mvc.Presentation.SpeakRenderingModel instead of the Sitecore.Mvc.RenderingModel.

The SpeakRenderingModel base class ensures that a js and a css file with the same name as the rendering is written as attributes for Require if such files exists. So no more requiring the scripts explicitly as long as you stick to the SPEAK naming convention.

_You should note that the SpeakRenderingModel  sets the require category to "client" which means that when you place your code in a different location you should either set Model.ScriptCategory to your own category name or make sure that the extra folder you patch into the resolveScript pipeline is in the "client" category._

Besides being derived from SpeakRenderingModel your new 1.2 model class is now auto-generated as a partial class by Rocks so you get all the fields from your rendering parameter strongly typed.

### _How to: Auto-generate your strongly typed View Model with Sitecore Rocks_

Sitecore Rocks has a new menu button for auto-generating your SPEAK view model from Rendering parameters.

First you of course need to create your rendering for your component.

Before you create your component then create a folder with the same name as the component that you are about to create.

![folder_for_speak_component](/wp-content/uploads/2014/10/folder_for_speak_component.png)_Always create a designated folder for each of your components. This is a lot cleaner than having a lot of different components cshtml, js and styling all in the same folder. So I repeat, always create a folder specific for your component._

When you've created your folder then right click on this in the Solution Explorer and click Add -> New Item -> SPEAK 1.2 -> SPEAK Component.

[![speak_templates_in_vs](/wp-content/uploads/2014/10/speak_templates_in_vs.png)](/wp-content/uploads/2014/10/speak_templates_in_vs.png)

Choose where to place the rendering item in Sitecore, also here remember to create a designated folder for your application and place the component rendering in a renderings folder beneath the application.

Now locate the generated rendering in Sitecore Explorer, right click and select _Create SPEAK Parameters template_ [![create_speak_parameters_template](/wp-content/uploads/2014/10/create_speak_parameters_template.png)](/wp-content/uploads/2014/10/create_speak_parameters_template.png)

Define your fields in the Parameters template and save.

[![edit_params_template](/wp-content/uploads/2014/10/edit_params_template.png)](/wp-content/uploads/2014/10/edit_params_template.png)

Now right click the rendering again and  choose Create SPEAK View Model.

[![create_speak_view_model](/wp-content/uploads/2014/10/create_speak_view_model.png)](/wp-content/uploads/2014/10/create_speak_view_model.png)

This will generate a partial class from the rendering parameters similar to the one shown below.

```c#
#pragma warning disable 1591

namespace TheComponent
{
  #region Designer generated code

  [System.Diagnostics.DebuggerStepThroughAttribute()]
  [System.CodeDom.Compiler.GeneratedCodeAttribute("SitecoreRocks", "1.0.0.0")]
  public partial class TheComponentRenderingModel : Sitecore.Mvc.Presentation.SpeakRenderingModel
  {
    [System.CodeDom.Compiler.GeneratedCodeAttribute("SitecoreRocks", "1.0.0.0")]
    public string AccessKey { get { return this.GetString("AccessKey"); } }

    [System.CodeDom.Compiler.GeneratedCodeAttribute("SitecoreRocks", "1.0.0.0")]
    public string Behaviors { get { return this.GetString("Behaviors"); } }

    [System.CodeDom.Compiler.GeneratedCodeAttribute("SitecoreRocks", "1.0.0.0")]
    public string Id { get { return this.GetString("Id"); } }

    [System.CodeDom.Compiler.GeneratedCodeAttribute("SitecoreRocks", "1.0.0.0")]
    public bool IsVisible { get { return this.GetBool("IsVisible"); } }

    [System.CodeDom.Compiler.GeneratedCodeAttribute("SitecoreRocks", "1.0.0.0")]
    public string Text { get { return this.GetString("Text"); } }

    [System.CodeDom.Compiler.GeneratedCodeAttribute("SitecoreRocks", "1.0.0.0")]
    public string Tooltip { get { return this.GetString("Tooltip"); } }
  }

  #endregion
}

#pragma warning restore 1591
```

If you change your Rendering parameters then simply click on _Create SPEAK view model_ again and the file will be updated.

I love this, no more plumbing code for getting the model and Sitecore hooked together, so easy.

### _Assigning model attributes_

In SPEAK v1.1 your fingers would almost be bleeding from typing by now but you hadn't even started to write the client side code. And then each time you should get or set a model attribute in JavaScript then you again had to call loosely typed get and set methods on the component model as shown in the code below.

```js
this.model.set("isVisible", this.model.get("text") != null && this.model.get("text").length > 0);
var isVisible = this.model.get("isVisible");
```

But now the model attributes client side are strongly typed as well. So in SPEAK 1.2 you would simply write

```js
this.IsVisible = Text != null && Text.length > 0;
var isVisible = this.IsVisible;
```

The old get and set calls still works fine so if you are already too used to these or are using some old code it is still all good.

It is so perfect that all the plumbing code now is generated. This really reduce the overhead when creating a new SPEAK component. But what is even better is..

## You can use TypeScript!!

Yeah.

If you want to use TypeScript for your component then add your new component using the template called _SPEAK Component_ and not the _SPEAK Component with Javascript_.

When you've made your rendering parameters template and generated your view model then you can right-click the rendering and pick _Create SPEAK TypeScript file_.

[![generate_speak_typescript_file](/wp-content/uploads/2014/10/generate_speak_typescript_file.png)](/wp-content/uploads/2014/10/generate_speak_typescript_file.png)

This will auto-generate a TypeScript class similar to the one shown below.

```js
import Speak = require("sitecore/shell/client/Speak/Assets/lib/core/1.2/SitecoreSpeak");

class TheComponent extends Speak.ControlBase {
  // #region Public Properties
  // This region was generated by a tool. Changes to this region may cause incorrect behavior and will be lost if the code is regenerated.
  public IsVisible: boolean;
  public Text: string;
  public Tooltip: string;
  // #endregion

  initialize(options: ComponentOptions, app: Application, el: Element, sitecore: SitecoreSpeak) {
  }
}

Sitecore.component(TheComponent, "TheComponent");

```

If the rendering parameter template changes then you can just click _Create SPEAK TypeScript file_ again even though you probably have written your own code within the TypeScript file.

The generator will only replace the code within the Public properties region when the file exists already.

You also need to reference the Sitecore SPEAK typing in your project if you want to use TypeScript.

The typings definition file is called Sitecore.Speak.d.ts and is located under <website>\\sitecore\\shell\\client\\Speak\\Assets\\lib\\core\\1.2

You can either reference it from each of your TypeScript files by placing a reference in the top of the file as shown below.

```js
/// <reference path="../../../../../sitecore/shell/client/Speak/Assets/lib/core/1.2/Sitecore.Speak.d.ts "/>
```

But this can be a bit messy and redundant depending on where you have placed your project. The relative path can be quite long. Another approach is to simply add the existing file to your project and voila you have type safety without referencing the file everywhere in your code.

[![speak_typescript_typing](/wp-content/uploads/2014/10/speak_typescript_typing.png)](/wp-content/uploads/2014/10/speak_typescript_typing.png)

On the above image I simply placed the typings file in the root of the project but it can be placed anywhere within the project. Note that this approach will not get the file updated when Sitecore is updated.

### _Do not fear TypeScript_

You should not fear TypeScript, if you are a C# developer you should rather embrace it. The extended JavaScript syntax is easy to learn and there is a lot of great material out there so Google will help you to get started and you will be coding TypeScript within minutes. The learning curve is not steep at all. You should also read my post on [3 reasons why TypeScript is brilliant](/3-reasons-typescript-brilliant/ "3 reasons why TypeScript is brilliant") if you are not completely convinced yet.

There are of course more good stuff in the new version of SPEAK so more posts coming up. It is so much quicker to get started and the amount of plumbing code required to get things running is reduced dramatically.

My next post on SPEAK will introduce the bootstrap component library made by Jakob Christensen which really shows how relative easy it is to make your own component libraries.
