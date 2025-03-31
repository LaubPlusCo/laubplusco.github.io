---
author: alc
category:
  - introducing-speak-components
  - sitecore
  - sitecore-7.1
  - speak
date: "2013-12-10T18:09:55+00:00"
guid: http://laubplusco.net/?p=296
title: 'Part 2: Introducing Sitecore SPEAK Components'
url: /creating-custom-sitecore-speak-components/

---
_This post is part 2 in a series showing how to create custom [Sitecore SPEAK components](/category/speak/introducing-speak-components/ "Sitecore SPEAK components")._

In this post I will show how to setup a custom SPEAK component in Sitecore. It is actually just setting up a rendering in Sitecore and then creating a rendering parameters template for it which is actually just a template item residing beneath the rendering item.

## Registering a SPEAK component in Sitecore

To do this we do not need to enter Sitecore at all, what we need to do is to use Sitecore Rocks as mentioned in [part 1](/introducing-sitecore-speak-components-some-basics/ "Introducing Sitecore SPEAK Components"). To make it even more easy then scope the Sitecore Explorer to the core database in which you want to register the control.

[![rocks_scope_to_this](/wp-content/uploads/2013/12/rocks_scope_to_this.png)](/wp-content/uploads/2013/12/rocks_scope_to_this.png)

All SPEAK applications should be self-contained when following the general SPEAK architecture which I think is one of the most significant architectural improvements to Sitecore which SPEAK introduces and Rocks makes possible.

So first we create a new SPEAK application to which the component belong and then we'll create a _Speak-RenderingFolder_ item under the application item.

[![renderings_folder](/wp-content/uploads/2013/12/renderings_folder.png)](/wp-content/uploads/2013/12/renderings_folder.png)

In this folder we create a new _View Rendering_ item.

[![view rendering](/wp-content/uploads/2013/12/view_rendering.png)](/wp-content/uploads/2013/12/view_rendering.png)

And then we'll set the path to a MVC Razor view, more about how these look in the next post in this series.

[![view rendering path](/wp-content/uploads/2013/12/view_rendering_path.png)](/wp-content/uploads/2013/12/view_rendering_path.png)

Next step is to define which rendering parameters are applicable for this rendering. To do this create a template item beneath the newly created rendering item andthen add the desired parameters as fields on the template.

[![rendering parameters template](/wp-content/uploads/2013/12/rendering_parameters_template.png)](/wp-content/uploads/2013/12/rendering_parameters_template.png)

Optionally set the desired bindmode for each parameter.

[![Bindmodes](/wp-content/uploads/2013/12/Bindmodes.png)](/wp-content/uploads/2013/12/Bindmodes.png)

The bind modes are not really documented in the documentation available on SDN.

But a bit of investigation quickly shows that bindmode=read means that you cannot set the parameter on the rendering in Rocks. If anyone can provide a clear written description of bindmodes please write a comment - and Sitecore, please update documention.

Then go back to the Rendering Item and set the Rendering parameters item to the template item you created.

[![rendering parameters field](/wp-content/uploads/2013/12/rendering_parameters_field.png)](/wp-content/uploads/2013/12/rendering_parameters_field.png)

Now when the rendering is added to the presentation of an item it can be opened in Rocks and the parameters are shown as properties.

[![rendering parameters on rendering](/wp-content/uploads/2013/12/rendering_parameters_on_rendering.png)](/wp-content/uploads/2013/12/rendering_parameters_on_rendering.png)

If the component which you are creating inherits from one of the base components and is defined using one of these on the client in JS (see [part 1](/introducing-sitecore-speak-components-some-basics/ "Introducing Sitecore SPEAK Components")) then you should let the template item inherit from the desired base template.

That was it now just a quick summary:

- Create custom SPEAK component renderings which are application specific as descendants to the application item.
- Rendering parameters in Sitecore are actually just a good old template which sets the properties for a rendering when it is opened in Rocks.
- Let the rendering parameters template derive from one of the predefined base templates in SPEAK if the component type is created using the [createBaseComponent](/introducing-sitecore-speak-components-some-basics/ "Introducing Sitecore SPEAK Components") function.
- Parameters can have different bind modes.

The next post will show how to write the Razor view for the SPEAK components and the final post in this series will show a full example of creating a SPEAK control from scratch.
