---
author: alc
category:
  - pipelines
  - signalr
  - sitecore
  - sitecore-7.1
  - speak
cover:
  alt: awesomechat
  image: /wp-content/uploads/2014/02/awesomechat.png
date: "2014-02-03T13:31:40+00:00"
guid: http://laubplusco.net/?p=697
title: SignalR component for SPEAK in Sitecore 7.1
url: /signalr-component-sitecore-speak/

---
Following [my blog post from earlier this week](/using-signalr-speak-application/ "Using SignalR in a SPEAK application") where I showed how to make SignalR work in Sitecore SPEAK, I thought of another solution to the same issue which is much more streamlined with SPEAK.

The issue which I hacked my way through was that the RequireJS implementation in SPEAK requires that a JavaScript file ends with .js if it does not then .js will be added automatically.

SignalR dynamically generates some JavaScript defining the hubs found in your solution. This script resides at _<website>/signalr/hubs_ with no .js postfix.

To solve this my previous "fix" added a file to the solution called hubs.js which placed a script reference to /signalr/hubs in the html head section. This then lead to another issue regarding the order in which the JavaScript was loaded into the DOM requiring one to use a timeout to ensure that the signalr hubs script was loaded before calling any of the hubs.

So now instead of this messy approach I came up with something a bit more clean using RequireJS.

## Fixing the missing .js in SignalR for good

First I added a processor called ResolveSignalrHubs in the resolveScript SPEAK pipeline (use patching in a separate include config file, do not write it directly as I did here).

```xhtml
  <speak.client.resolveScript>
    <processor type="Sitecore.Resources.Pipelines.ResolveScript.Main, Sitecore.Speak.Client" />
    <processor type="Sitecore.Resources.Pipelines.ResolveScript.Rule, Sitecore.Speak.Client" />
    <processor type="SpeakSignalR.ResolveSignalrHubs, SpeakSignalR" />
    <processor type="Sitecore.Resources.Pipelines.ResolveScript.ResolveBaseComponent, Sitecore.Speak.Client" />
    <processor type="Sitecore.Resources.Pipelines.ResolveScript.Controls, Sitecore.Speak.Client">
      <sources hint="raw:AddSource">
        <source folder="/Components/SpeakSignalR/Scripts" deep="true" category="signalr" pattern="*.js" />
        <source folder="/Components/SpeakSignalR/SignalR" deep="true" category="client" pattern="*.js" />
        <source folder="/sitecore/shell/client/Speak/Assets" deep="true" category="assets" pattern="*.js" />
...
```

The processor handles all requests made to the SPEAK customhandler which starts with _signalr/hubs.js_

Then it makes a server request to _/signalr/hubs,_ reads the response and encapsulate the returned script in a _define_ which requires the needed jQuery.signalr script.

```c#
  public class ResolveSignalrHubs : ResolveScriptProcessor
  {
    public override void Process(ResolveScriptArgs args)
    {
      Assert.ArgumentNotNull(args, "args");
      if (!args.FileName.StartsWith("signalr/hubs.js", StringComparison.InvariantCultureIgnoreCase))
        return;
      args.AbortPipeline();
      var urlString = new UrlString(WebUtil.GetScheme()+ "://" + WebUtil.GetHostName() + "/signalr/hubs");
      var request = WebRequest.Create(urlString.ToString());
      var responseStream = request.GetResponse().GetResponseStream();
      Assert.IsNotNull(responseStream, "Could not read SignalR hubs script");
      var script = new StreamReader(responseStream).ReadToEnd();
      script = RequireSignalR(script);
      var bytes = Encoding.UTF8.GetBytes(script.ToCharArray());
      args.Content = new MemoryStream(bytes);
    }

    private string RequireSignalR(string script)
    {
      return "define([\"/-/speak/v1/signalr/jquery.signalR-2.0.2.min.js\"], function() {"
        + script + "});";
    }
  }
```

And now by accessing _/-/speak/v1/signalr/hubs.js_ we get the dynamically generated SignalR hubs script encapsulated in a define stating that the function requires the jquery.signalr script.

```js
define(["/-/speak/v1/signalr/jquery.signalR-2.0.2.min.js"], function() {/*!
 * ASP.NET SignalR JavaScript Library v2.0.2
 * http://signalr.net/
 *
 * Copyright Microsoft Open Technologies, Inc. All rights reserved.
 * Licensed under the Apache 2.0
 * https://github.com/SignalR/SignalR/blob/master/LICENSE.md
 *
 */

/// <reference path="..\..\SignalR.Client.JS\Scripts\jquery-1.6.4.js" />
/// <reference path="jquery.signalR.js" />
(function ($, window, undefined) {
    /// <param name="$" type="jQuery" />
    "use strict";

    if (typeof ($.signalR) !== "function") {
        throw new Error("SignalR: SignalR is not loaded. Please ensure jquery.signalR-x.js is referenced before ~/signalr/js.");
    }

    var signalR = $.signalR;
...
```

This solves the issue, no need for a timeout anymore and all is good.

Now we are ready to really SPEAKify SignalR.

## Encapsulating SignalR in SPEAK

Next I've created a SPEAK component which wraps the SignalR connection to one specific hub, several SignalR components can exist on the same SPEAK page if several hubs exists.

```c#
@using Sitecore.Mvc
@using Sitecore.Mvc.Presentation
@using Sitecore.Web.UI.Controls.Common.UserControls
@model RenderingModel
@{
  var rendering = Html.Sitecore().Controls().GetUserControl(Model.Rendering);
  rendering.Class = "sc-SignalR";
  rendering.Requires.Script("client", "SignalR.js");
  rendering.SetAttribute("data-sc-hubname", rendering.GetString("HubName", "hubName", string.Empty));
  var htmlAttributes = rendering.HtmlAttributes;
}
<div @htmlAttributes>
</div>
```

As a rendering parameter the component takes the name of the SignalR hub otherwise the rendering parameters simply inherits from ComponentBase.

The model has a start function which starts up the connection to the hub and then triggers an event when the connection to the server is setup telling that it is ready for use.

```js
define(["jquery", "sitecore", "/-/speak/v1/signalr/hubs.js"], function ($, Sitecore) {
  "use strict";
  var model = Sitecore.Definitions.Models.ComponentModel.extend(
    {
      initialize: function (options) {
        this._super();
        this.set("hubName", undefined);
        this.set("started", false);
      },
      setHub: function (hubName) {
        this.set("hubName", hubName);
        if (hubName === "" || hubName === undefined)
          throw "Missing hubName";
        var hub = $.connection[hubName];
        if (hub === null)
          throw "Could not find hub with name " + hubName;
        this.set("hub", hub);
      },
      addClientHandler: function (broadcast, handler) {
        if (this.get("started"))
          throw "You cannot add a client handler after hub connection started";
        var hub = this.get("hub");
        hub.client[broadcast] = handler;
      },
      start: function () {
        if (this.get("started"))
          return;
        var hub = this.get("hub");
        var m = this;
        $.connection.hub.start(this).done(function () {
          m.set("server", hub.server);
          m.set("started", true);
          m.trigger("ready");
        });
      }
    }
  );

  var view = Sitecore.Definitions.Views.ComponentView.extend(
    {
      initialize: function () {
        this._super();
        var hubName = this.$el.attr("data-sc-hubname");
        this.model.setHub(hubName);
      }
    }
  );

  Sitecore.Factories.createComponent("SignalR", model, view, ".sc-SignalR");
});
```

The reason why I do not start the connection right away is because SignalR will require you to set the client broadcast receiver handlers before starting the connection. This feels a bit like a bug/lack in SignalR which can leave you with a lot of debugging headache if you are not aware of this specific sequence in which things need to be coded since there are no warnings or anything, just code which does not work.

## Example usage

First we add the SignalR component to a SPEAK page and type in the name of the hub implementation which we want to wrap in this SignalR component.

[![SignalR Rendering Item](/wp-content/uploads/2014/02/SignalRRenderingItem.png)](/wp-content/uploads/2014/02/SignalRRenderingItem.png)

Then in the PageCode we can use the component like this:

```js
define(["sitecore"], function(Sitecore) {
  var PageCode = Sitecore.Definitions.App.extend({
    initialized: function() {
      var app = this;
      this.ChatHub.addClientHandler("receiveMessage",
        function(userName, message, timestamp, id) {
          app.MessagesDataSource.add(
          {
            userName: userName,
            message: message,
            timeStamp: timestamp,
            itemId: id
          });
        });

      this.ChatHub.on("ready", function () {
        app.MessageText.viewModel.$el.keydown(function(e) {
          if (e.keyCode != 13) return;
          app.sendMessage();
        });
        app.on("button:send", function () {
          app.sendMessage();
        }, this);
      });

      this.ChatHub.start();
    },
    sendMessage: function() {
      var text = this.MessageText.get("text").trim();
      if (text === "")
        return;
      var userName = this.ChatContext.get("userName");
      this.ChatHub.get("server").send(userName, text);
      this.MessageText.set("text", "");
    }
  });
  return PageCode;
});
```

Server side we of course also need to implement a _Microsoft.AspNet.SignalR.Hub_ etc.

## SPEAK and SignalR

Compared with having all the code for hooking up SignalR in the PageCode directly as I showed in a [previous post](/using-signalr-sitecore-speak-application/ "Using SignalR in a Sitecore 7.1 SPEAK application") then this is much easier to read and the SignalR component can be re-used in any SPEAK/SignalR implementation.

Using SignalR with SPEAK is extremely powerful. Anyone who have played around with SignalR will know that this technology has a huge potential and at the same time is really easy to use.

Imagine SPEAK and SignalR combined with dynamically generated TypeScript header files for getting strongly typed JavaScript. Creating a framework for this would be among, if not the most, powerful framework for creating heavy client side business applications.

Generating TypeScript headers for SignalR can be done using T4 (see for example this [T4 template on gitHub](https://gist.github.com/robfe/4583549)). So all we really need is TypeScript for SPEAK.

I will release this SignalR component along with my configured pipelines component as nuGet packages as soon as I have the time for it.

And as with the pipelines I hope Sitecore will endorse these approaches and make it a part of the SPEAK API in future versions. They probably already have a ton of other cool features lined up for 7.2 but I hope there will be room for SignalR as well, if not for 7.2 then later.

I will post a full example on how to use this SignalR component combined with a configured client side pipeline soon.
