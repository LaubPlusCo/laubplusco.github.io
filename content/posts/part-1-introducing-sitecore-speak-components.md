---
author: alc
category:
  - introducing-speak-components
  - sitecore
  - sitecore-7.1
  - speak
date: "2013-12-03T18:08:24+00:00"
guid: http://laubplusco.net/?p=303
title: 'Part 1: Introducing Sitecore SPEAK Components'
url: /introducing-sitecore-speak-components-some-basics/

---
_This post is part 1 in a series showing how to create custom [Sitecore SPEAK components](/category/speak/introducing-speak-components/ "Sitecore SPEAK components")._

It is time for Sitecore developers to learn Javascript and become all around web developers not just hiding back in the save haven of the server. Like it or not this is the way things are going. Looking on the bright side with todays frontend technologies it is almost the same to work on the client as on the server.

The new Sitecore SPEAK uses a ton of new things seen from my perspective as a C# developer, Backbone, Knockout, Require etc. all common JS libraries which all sound very hipster and for me something I only came across before when reading random startup blog posts.

I spend quite some time now trying to learn what all of these frameworks do and I am beginning to somewhat like them.

The new SPEAK API really bring Sitecore well into the future compared to the now rather old SheerUI which otherwise really has stood the test of time.

Besides using all of these client side frameworks and wrapping their functionality into a nice reusable Sitecore UI framework I think the main new concept is that control can move from the server to the client.

SheerUI used a continuation flow where the client initiates a command on the server whereafter the server takes full control of the following flow. This approach kept the client rather light and allowed server side pipelines etc. to talk back to the client. Looking at the old sitecore.js there is a switch-case statement from hell for handling all possible server replies and a wildcard "eval" case which just handles any other Javascript that the server sends.

This approach has been fine for many years but now modern browser technologies and html 5 allow the client to take the control. The server then ideally only needs to be called when CRUD operations are required.

So in Sitecore SPEAK the continuation flow is now controlled by the client and not the server. SPEAK also introduce the good old pipeline principle client side allowing code to be run in Javascript pipelines.

Before continuing I would recommend everyone to see the videos posted by the Rocks guy here: [http://www.youtube.com/user/JakobHChristensen?feature=watch](http://www.youtube.com/user/JakobHChristensen?feature=watch)

To get started on the new Sitecore SPEAK you'll need to use Sitecore Rocks it is still not possible to use the content editor. I am actually a huge fan of forcing developers into this especially with the new 1.0 release of Rocks. I myself was a bit slow getting started on Rocks and leaving the comfort of the content editor but now when I am used to Rocks I won't go back unless required.

What about the previous version of SPEAK then? Well that was truely something completely different and I am glad that Sitecore retried and made this new version.

Moving the SPEAK applications to the core database instead of having them in the master database under /sitecore/system/modules/speak really makes SPEAK a real part of the product. I still miss a lot of stuff in the new SPEAK version, for example a fixed REST server api but I am sure that it will come some time soon. At least the documentation is extremely good which some might say is new for Sitecore.

## Creating a component

Now to some code.

A SPEAK page uses a SPEAK layout which then is built up of SPEAK components. A SPEAK component is actually just a Razor view setup as a rendering in Sitecore with a parameters template. _More about this in the next post._

Each component type is defined in Javascript as a model and a view on the client and is then accessible in the PageCode Javascript.

There are two ways of defining a component in Javascript.

**Using an existing base component from SPEAK**

This means that your component is derived from a predefined base component.

To do this call the Sitecore.Factories.createBaseComponent function. This function takes the name of the component type, the name of the base component to use, the selector and optional a collection of attributes as arguments. You can also add an array of events for the view as arguments.

```js
define(["sitecore"], function (_sc) {
  _sc.Factories.createBaseComponent({
    name: "NAMEOFCOMPONENT",
    base: "BASETOINHERITFROM",
    selector: "SELECTOR",
    events:
  {
    "click .SOME-SELECTOR": "clickedSelector",
  },
    attributes: [
        { name: "ATTRIBUTENAME", value: "$el.data:sc-DATAATTRIBIUTE" }
    ],
    initialize: function () {

    },
    someFunction: function (param) {
      alert(param);
    },
    clickedSelector: function (event) {
      // do something with event.target
    }
  });
});
```

The initialize method is called when a component of this type is created. If you want to call the someFunction function it is neither on the view nor the model but on the viewModel. So to call a custom function from a page code you would in this case have to call it on the viewModel object.

```js
this.CustomComponent.viewModel.someFunction("hey")
```

There exists a number of base components and there will probably be more added during time. These can all be seen in the core database under /sitecore/client/Speak/Templates/Rendering Parameters.

[![SPEAK BaseComponents](/wp-content/uploads/2013/12/BaseComponents.png)](/wp-content/uploads/2013/12/BaseComponents.png)

When using the createBaseComponent function the rendering parameters template used by your custom component rendering must inherit from the base component template. _More on this in [part 2](/creating-custom-sitecore-speak-components/ "Part 2: Introducing Sitecore SPEAK Components")._ **Creating a component from scratch**

The base components all have predefined attributes which does not necessarily apply for all component s, this  could be for datasources or components which just holds on to some context attributes.

Or it could be that you simply want more control than what you get with a base component.

In that case you'll create a component using the Sitecore.Factories.createComponent function which takes the name of the component type, the model definition, the view definition and the selector as arguments.

```js
define(["sitecore"], function ($, sitecore) {
    "use strict";
    var model = sitecore.Definitions.Models.ComponentModel.extend(
	{
		initialize: function (attributes) {
			this._super();
			this.set("ATTRIBUTENAME", "");
			this.on("change:ATTRIBUTENAME ", this.SOMEFUNCTION, this);
		},
		SOMEFUNCTION: function() {
			// DO SOMETHING
		}
	});

    var view = sitecore.Definitions.Views.ComponentView.extend(
      {
        initialize: function (options) {
          this._super();
          this.model.set("ATTRIBUTENAME", this.$el.attr("data-sc-SOMEDATAATTRIBIUTE"));
        }
      }
    );
    sitecore.Factories.createComponent("[COMPONENT NAME]", model, view, "SELECTOR");
});
```

If you're still completely on the server side and hit the very steep learning curve of Javascript then think of SPEAK components as being somewhat analogous to a normal class in C#. I know, Javascript has real objects and almost a class hierarchy using prototyping but coming from a real object oriented language this seems very hacky. What SPEAK offers here is actually prototyping done easy.

There really is a lot of new stuff to learn when using Sitecore SPEAK but climbing the steep learning curve is time well spent. This post only cover some narrow aspects and the code shown here could use three new posts of explanation, so more coming up soon and probably also updates to this one.

Please write a comment if you would like to share your experiences or opinions about the new Sitecore SPEAK API.
