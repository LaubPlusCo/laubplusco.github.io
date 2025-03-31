---
author: alc
category:
  - sitecore
  - sitecore-7.1
  - speak
cover:
  alt: json
  image: /wp-content/uploads/2014/01/json.png
date: "2014-01-26T18:51:07+00:00"
guid: http://laubplusco.net/?p=634
title: Creating a simple Sitecore SPEAK JSON datasource
url: /creating-simple-sitecore-speak-json-datasource/

---
In SPEAK a list  is fed with data through a data source. For example to get data into a ListControl the _Items_ rendering parameter on the ListControl has to be bound to a collection on a datasource.

When the items model attribute changes, that is when the datasource changes, then the ListControl automatically refreshes and shows the new data.

SPEAK ships with 3 different data sources which are all made for Sitecore items.

[![DataSources](/wp-content/uploads/2014/01/DataSources.png)](/wp-content/uploads/2014/01/DataSources.png)

These are fine for working with Sitecore items but I really miss a simple data source just for JSON data.

So during the [Sitecore Hackathon](http://sitecorehackathon.org/) we needed exactly this for a chat application.

## Creating a JsonDataSource

First I created a custom SPEAK component with the following cshtml

```c#
@using Sitecore.Mvc
@using Sitecore.Mvc.Presentation
@using Sitecore.Web.UI.Controls.Common.UserControls
@model RenderingModel
@{
    var userControl = Html.Sitecore().Controls().GetUserControl(Model.Rendering);
    userControl.Requires.Script("chat", "jsondatasource.js");
    userControl.Attributes["type"] = "text/x-sitecore-jsondatasource";
    userControl.DataBind = "Json: json";
    var htmlAttributes = userControl.HtmlAttributes;
}
<script @htmlAttributes>
</script>
```

And the following client side implementation.

```js
define(["jquery", "sitecore"], function ($, Sitecore) {
  "use strict";

  var model = Sitecore.Definitions.Models.ComponentModel.extend(
    {
      initialize: function (attributes) {
        this._super();
        this.set("json", null);
      },
      add: function (data) {
        var json = this.get("json");
        if (json === null)
          json = new Array();

        // this is done because array.push changes the array to an object which then do no work on the SPEAK listcontrol.
        var newArray = new Array(json.length + 1);
        for (var i = json.length - 1; i >= 0; i--)
          newArray[i + 1] = json[i];
        newArray[0] = data;
        this.set("json", newArray);
      }
    }
  );

  var view = Sitecore.Definitions.Views.ComponentView.extend(
    {
      initialize: function () {
        this._super();
        this.model.set("json", null);
      }
    }
  );

  Sitecore.Factories.createComponent("JsonDataSource", model, view, "script[type='text/x-sitecore-jsondatasource']");
});
```

This implementation is extremely simple.

We bind the model attribute json with the rendering parameter Json. Then we have an add function for pushing new objects into the json array.

Notice that we reconstruct the json array on each add. The reason for this is a bit strange. Usually in JavaScript adding to an array is easy. Just call <array>.push(newObject) to add the object to the end of the array. But calling this method changes the array into an object with array as prototype.

For some reason the SPEAK ListControl do not like this. We need the array to be a real array, I cannot give you a clear explanation for this but it took quite some time and headache to figure out.

Also the add method adds to the beginning of the array so newest added is on top.

Next we need to register the rendering in Sitecore. First we create a Speak-Renderings folder in the application where we need the datasource and then we create a View Rendering item in this folder called JsonDataSource.

[![Json DataSource View Rendering](/wp-content/uploads/2014/01/JsonDataSourceViewRendering.png)](/wp-content/uploads/2014/01/JsonDataSourceViewRendering.png)

Next thing is to create the rendering parameters. This is done by creating a template item beneath the JsonDataSource view rendering item.

[![Json Data Source RenderingParameters](/wp-content/uploads/2014/01/JsonDataSourceRenderingParameters.png)](/wp-content/uploads/2014/01/JsonDataSourceRenderingParameters.png)

On the template we create one field called Json with bindmode=read and then we set the ComponentBase rendering parameters template as a base template.

Finally we point the rendering parameters field on the view rendering to the rendering parameters template. [![SetRenderingParametersTemplate](/wp-content/uploads/2014/01/SetRenderingParametersTemplate.png)](/wp-content/uploads/2014/01/SetRenderingParametersTemplate.png)

Now we can bind a ListControl items parameter to the Json parameter of the JsonDataSource.

[![Bind DataSource On List](/wp-content/uploads/2014/01/BindDataSourceOnList.png)](/wp-content/uploads/2014/01/BindDataSourceOnList.png)

That was almost it.

_**Important**_

There is one small glitch though which I hope will be fixed in later versions of SPEAK.

The ListControl expects that all the json objects have a property called _**itemId**_, if it does not then it ignores the row.

Just add a property called itemId containing any value and it works just fine.

In a later post I will show some more about the ListControl and how it is configured.

Here is a working example for calling add on the datasource:

```js
      Sitecore.chat.MessagesDataSource.add(
      {
        userName: userName,
        message: message,
        timeStamp: timestamp,
        itemId: id
      });
```

 _Note that it is also possible simply to add JSON directly to the items attribute on a ListControl but I prefer this binding approach for separating responsibility._
