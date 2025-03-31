---
author: alc
category:
  - signalr
  - sitecore
  - sitecore-7.1
  - speak
date: "2014-01-26T16:52:45+00:00"
guid: http://laubplusco.net/?p=621
title: Using SignalR in a Sitecore 7.1 SPEAK application
url: /using-signalr-sitecore-speak-application/

---
During the [Sitecore Hackathon](http://sitecorehackathon.org/) we decided to implement a chat app for Sitecore editors so it is possible to send text messages to other online editors.

To do this we wanted to use [SignalR](http://signalr.net/) and SPEAK for the UI part. Setting up SignalR and creating the SPEAK layout was done quickly but it quickly became obvious that using the two together could be trickier than anticipated.

The issue was not with SPEAK itself or SignalR but with RequireJS and the order in which Javascript is loaded.

SignalR creates a piece of JavaScript dynamically based on your hub definitions. This JavaScript is then accessed on _<website>/signalr/hubs_ notice that there is no .js postfixed on the path.

This is not good because the SPEAK utilization of RequireJS expects a .js filename on JavaScript files.

Well some hacking was needed, this is not pretty but it works.

First I've created a js file called hubs.js

```js
// Allow to reference /signalr/hubs even though it is not postfixed with .js
var head = document.getElementsByTagName('head')[0];
var script = document.createElement('script');
script.type = 'text/javascript';
script.src = "/signalr/hubs";
head.appendChild(script);
```

This small script places a script tag in the page head section which points to /signalr/hubs

Then I've added a source folder in config for the speak.client.resolveScript pipeline.

```xhtml
<pipelines>
  <speak.client.resolveScript>
    <processor type="Sitecore.Resources.Pipelines.ResolveScript.Controls, Sitecore.Speak.Client">
      <sources hint="raw:AddSource">
        <source patch:after="source[last()]" folder="/hackathon/messaging/scripts" deep="true" category="messaging" pattern="*.js" />
      </sources>
    </processor>
  </speak.client.resolveScript>
</pipelines>
```

In the SPEAK page code I then required the scripts needed by SignalR

```js
define(["sitecore", "/-/speak/v1/messaging/jquery.signalR-2.0.2.min.js", "/-/speak/v1/messaging/hubs.js"], function (Sitecore) {
  var PageCode = Sitecore.Definitions.App.extend({
    initialized: function () {
....
```

That was almost it. All the scripts are now being loaded without having to create a SPEAK component for holding the reference or use the SPEAK-Script reference items (which are not working properly in the current version of SPEAK).

The last thing is a bit of a hack. To ensure that the /signalr/hubs script has been loaded into the DOM we make a check to see if the SignalR connection contains the hub we expect it to contain. If it does not then we wait 100 milliseconds and check again.

When the object is no longer undefined then we start up the hub and configure the connection.

```js
    initialized: function () {
      Sitecore.chat = this;
      this.waitForHubs();
    },
    waitForHubs: function() {
      if ($.connection.chatHub === undefined) {
        setTimeout(this.waitForHubs, 100);
      } else {
        Sitecore.chat.initializeSignalR();
      }
    },
    initializeSignalR: function() {
      var chat = $.connection.chatHub;
      chat.client.receiveMessage = Sitecore.chat.receiveMessage;
      chat.client.userLoggedIn = Sitecore.chat.userLoggedIn;
      chat.client.userLoggedOut = Sitecore.chat.userLoggedOut;

      $.connection.hub.start().done(function () {
        Sitecore.chat.MessageText.viewModel.$el.keydown(function (e) {
          if (e.keyCode != 13) return;
          Sitecore.chat.sendMessage(chat);
        });
        Sitecore.chat.SendButton.on("button:send", function () {
          Sitecore.chat.sendMessage(chat);
        });
      });
    },
```

I will write some more posts about the chat app and release it on market place soon. It needs a whole lot of polishing though, time ran out quickly at the Hackathon and this hack took some hours to come up with. Anyway, here is a small first version screenshot as a teaser. [![Awesome Chat for Sitecore](/wp-content/uploads/2014/01/chatapp.png)](/wp-content/uploads/2014/01/chatapp.png)
