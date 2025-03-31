---
author: alc
category:
  - introducing-speak-components
  - sitecore
  - sitecore-7.1
  - speak
date: "2013-12-17T18:17:33+00:00"
guid: http://laubplusco.net/?p=325
title: 'Part 4: Introducing Sitecore SPEAK Components'
url: /part-4-introducing-sitecore-speak-components/

---
This post is the fourth and final in a series about [Sitecore SPEAK components](/category/speak/introducing-speak-components/ "Sitecore SPEAK components").

Now we have seen how a SPEAK component is built in part [1](/introducing-sitecore-speak-components-some-basics/ "Part 1: Introducing Sitecore SPEAK Components"), [2](/creating-custom-sitecore-speak-components/ "Part 2: Introducing Sitecore SPEAK Components"), and [3](/part-3-introducing-sitecore-speak-components/ "Part 3: Introducing Sitecore SPEAK Components"). In this post I will show a full example of a really simple custom control.

## Why create a Custom SPEAK component anyway?

Sitecore delivers 70++ SPEAK controls and components which should cover most needs, but in some cases you might want something completely domain specific in your SPEAK application which cannot be achieved with the standard controls.

Always look for a standard control first to get the full benefit of the SPEAK api only resort to creating your own control as a last option.

## Creating the SPEAK control

Okay this control is going to be really kind of pointless it is purely an example. The control will consist of a text which is set as a parameter on the control and a counter for counting how many times it has been clicked. When it has been clicked 2 times the text is reversed and the background color of the control is changed.

**_Step 1:_**

First we need to create a project in Visual Studio for the control. The project should just be an empty MVC3 or higher web application project.

In my example I have placed the project under /Components/SpeakExample/ and to make this work the project also needs to contain a web.config file.

```xhtml
<?xml version="1.0"?>
<configuration>
  <configSections>
		<sectionGroup name="system.web.webPages.razor" type="System.Web.WebPages.Razor.Configuration.RazorWebSectionGroup, System.Web.WebPages.Razor, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35">
			<section name="host" type="System.Web.WebPages.Razor.Configuration.HostSection, System.Web.WebPages.Razor, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" requirePermission="false" />
			<section name="pages" type="System.Web.WebPages.Razor.Configuration.RazorPagesSection, System.Web.WebPages.Razor, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" requirePermission="false" />
		</sectionGroup>
  </configSections>

	<appSettings>
		<add key="webpages:Version" value="2.0.0.0" />
		<add key="ClientValidationEnabled" value="true" />
		<add key="UnobtrusiveJavaScriptEnabled" value="true" />
	</appSettings>

	<system.web.webPages.razor>
		<host factoryType="System.Web.Mvc.MvcWebRazorHostFactory, System.Web.Mvc, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" />
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

 [![web config in vs](/wp-content/uploads/2013/12/web_config_in_vs.png)](/wp-content/uploads/2013/12/web_config_in_vs.png)

This file also exists in the /sitecore/client/shell folder for making the out-of-the-box controls work. Sitecore actually made a folder under /sitecore/client/shell/speak/yourapps for custom SPEAK applications but I don't like to have custom code within the /sitecore folder.

Reference the following libraries from the /bin folder:

- Sitecore.Kernel
- Sitecore.Mvc
- Sitecore.Speak.Mvc

Now create a folder called Controls and in this create another folder for the control called ExampleControl

[![example control](/wp-content/uploads/2013/12/example_controlpng.png)](/wp-content/uploads/2013/12/example_controlpng.png)

When the folder is created, then add a cshtml file, a js file and a less/css file all called ExampleControl.

Now open the cshtml file and insert the following mark up and code, (see [part 3](/part-3-introducing-sitecore-speak-components/ "Part 3: Introducing Sitecore SPEAK Components") simple approach):

```c#
@using Sitecore.Mvc.Presentation
@using Sitecore.Mvc
@using Sitecore.Web.UI.Controls.Common.UserControls
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

Notice the Knockout data-bind attributes which binds the model attributes to the markup.

And for some styling I did this dirty stuff here:

```less
.example-control {
    width: 100%;
    padding: 20px 20px;
    background-color: red;
}

.example-text, .example-count {
    text-align: center;
    width: 100%;
    display: block;
    clear: both;
}

.example-text {
    font-size: 1.5em;
    background-color: #eee;
}

.example-count {
    margin-top: 20px;
    font-size: 3.5em;
    font-weight: bolder;
}
```

 **_Step 2:_**

Now go to Sitecore Rocks and register the cshtml as a rendering. Create the rendering in a rendering folder beneath your custom SPEAK application to keep it confined (see [part 2](/creating-custom-sitecore-speak-components/ "Part 2: Introducing Sitecore SPEAK Components")).

Add a parameters template as a child item to the rendering and add a parameter called DisplayedText.

[![ExampleControl rendering](/wp-content/uploads/2013/12/examplecontrol_rendering.png)](/wp-content/uploads/2013/12/examplecontrol_rendering.png)

Now set the parameters template field on the rendering item to the newly created sub item (see [part 2](/creating-custom-sitecore-speak-components/ "Part 2: Introducing Sitecore SPEAK Components")).

**_Step 3:_**

Now to the JavaScript, open the ExampleControl.js file and create a base component which inherits from BlockBase (see [part 1](/introducing-sitecore-speak-components-some-basics/ "Part 1: Introducing Sitecore SPEAK Components")).

Create a model attribute for the DisplayedText parameter and another model attribute called count which we give the default value 0. Also add an event for whenever the selector example-text is clicked.

```js
define(["sitecore"], function (_sc) {
  _sc.Factories.createBaseComponent({
    name: "ExampleControl",
    base: "BlockBase",
    selector: ".example-control",
    events:
  {
    "click .example-text": "onTextClicked"
  },
    attributes: [
        { name: "displayedText", value: "$el.data:sc-displayedtext" },
        { name: "count", defaultValue: 0 }
    ],
    initialize: function () {
      this._super();
      this.green = false;
      this.model.on("change:count", this.onCountChanged, this);
      this.model.on("change:displayedText", this.onTextChanged, this);
      console.log(this);
    },
    onTextClicked: function (event) {
      this.model.set("count", this.model.get("count") + 1);
    },
    onCountChanged: function () {
      if (this.model.get("count") % 2 == 0) {
        this.green = !this.green;
        this.$el.css("background-color", this.green ? "lightgreen" : "red");
        this.model.set("displayedText", this.reverse(this.model.get("displayedText")));
      }
    },
    reverse: function (text) {
      return text.split("").reverse().join("");
    }
  });
});
```

In the initialize method we listen for when the attributes changes and then we create a method for handling the click event and another method for handling the change of count. The onTextClicked method simply counts up the count attribute with one whilst the onCountChanged method checks if the count modulus 2 is equal to zero. If this is the case the background color of the whole control element is changed and the text is reversed. **_Step 4:_**

Finally to test this ingenious control we insert it on the presentation details of an item in our SPEAK application in core.

First we need to create the SPEAK application. Create a node item called SpeakExamples and in this create another node item called Pages. Here we create an Speak-DashboardPage item called StartPage and beneath this we create a Speak-PageSettings Item. Under the PageSettings item we create a folder called Texts in which we add a Text item called Header.

[![Example SPEAK Application](/wp-content/uploads/2013/12/ExampleApp.png)](/wp-content/uploads/2013/12/ExampleApp.png)

In the Text field on the Header Text item we write SPEAK examples.

Now design the layout of the StartPage item and insert the following renderings. Where the HeaderText datasource is set to the newly created Text item.

[![StartPage SPEAK Layout](/wp-content/uploads/2013/12/StartPageLayout.png)](/wp-content/uploads/2013/12/StartPageLayout.png)

For more information about setting up a SPEAK application see the Sitecore Order Manager example and the [videos from the Rocks guy](http://www.youtube.com/user/JakobHChristensen?feature=watch).

The control now displays like this on the page:

[![ExampleControl](/wp-content/uploads/2013/12/ExampleControl.png)](/wp-content/uploads/2013/12/ExampleControl.png)

And when the click count modulo two is equal to zero:

[![Example ControlModulo Two](/wp-content/uploads/2013/12/ExampleControlModuloTwo.png)](/wp-content/uploads/2013/12/ExampleControlModuloTwo.png)

Voila.

Quite an overcomplicated approach to achieve this rather pointless functionality but I hope that it still serve as a good example showing how easy it is to set up a custom SPEAK component within a SPEAK application.

## Sum up

This post showed how to create a very simple custom SPEAK control. It is based on the previous posts shown in a series about [SPEAK components](/category/speak/introducing-speak-components/).

Please do not hesitate to write me or drop a comment if you have any questions / suggestions / corrections what so ever. I will just appreciate any feed back and thanks for reading this far.

The code can be downloaded from [this post with an action and condition introduced](/creating-custom-sitecore-speak-rule-conditions-actions/ "Creating custom Sitecore SPEAK rule Conditions and Actions").

Note that this post is based on the version of SPEAK which shipped with Sitecore 7.1.0 rev. 130926. The API is likely to change slightly so Sitecore updates which makes some of this post outdated might occur.
