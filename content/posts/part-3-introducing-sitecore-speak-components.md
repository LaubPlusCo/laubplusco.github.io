---
author: alc
category:
  - introducing-speak-components
  - sitecore
  - sitecore-7.1
  - speak
date: "2013-12-14T18:11:10+00:00"
guid: http://laubplusco.net/?p=323
title: 'Part 3: Introducing Sitecore SPEAK Components'
url: /part-3-introducing-sitecore-speak-components/

---
_This post is the third in a series about [Sitecore SPEAK components](/category/speak/introducing-speak-components/ "Sitecore SPEAK components")._ A SPEAK control rendering can be implemented in two ways.

1. **Simple approach**, the control consists of a Razor view, a Javascript file and potentially some styling.
1. **Advanced approach**, consist of the same as the simple approach with two C# classes and some extension method added.

If databound properties are inherited from a base component then using the Advanced Approach can save some plumbing code. But this is also at the cost of lower flexibility and readability.

Sitecore uses the Advanced approach for most of their controls. This makes sense since they are pre-compiled and part of the framework.

Using the advanced approach also enables the component to be either instantiated from an items presentation details in Sitecore or from within a Razor view. The simple approach only works for being set as a rendering on presentation details.

## Creating the SPEAK component in Visual Studio

When creating a new component keep all code within the same folder structure so the control code is nicely confined within its own namespace. This follows the same architectural princip as mentioned in [part 2](/creating-custom-sitecore-speak-components/ "Part 2: Introducing Sitecore SPEAK Components") stating that SPEAK applications and controls should be self-contained.

Create a folder for the control and then create a Razor view, a js file and a less/css file (last only if styling is needed).

[![Nice and confined control](/wp-content/uploads/2013/12/NiceAndConfinedControl1.png)](/wp-content/uploads/2013/12/NiceAndConfinedControl1.png)

Name all of the files with the control name. All of these files are needed for both the simple and advanced approach.

To make SPEAK able to find the js and css file using RequireJs you will need to add or patch in the following line to the Sitecore.Speak.config file located in the /Include folder.

```xhtml
      <speak.client.resolveScript>
        <processor type="Sitecore.Resources.Pipelines.ResolveScript.Main, Sitecore.Speak.Client" />
        <processor type="Sitecore.Resources.Pipelines.ResolveScript.Rule, Sitecore.Speak.Client" />
        <processor type="Sitecore.Resources.Pipelines.ResolveScript.ResolveBaseComponent, Sitecore.Speak.Client" />
        <processor type="Sitecore.Resources.Pipelines.ResolveScript.Controls, Sitecore.Speak.Client">
          <sources hint="raw:AddSource">
           ...
            <source folder="/Components/SpeakExamples/Controls" deep="true" category="speakexamples" pattern="*.js,*.css" />
           ...
          </sources>
        </processor>
      </speak.client.resolveScript>
```

Take note of the category attribute which is used in the SPEAK component when a file from that specific folder is required.

_The implementation of the client side code is briefly described in [part 1](/introducing-sitecore-speak-components-some-basics/ "Part 1: Introducing Sitecore SPEAK Components") and part 4 will show a complete example with working code._

### Simple Approach

The simple approach is to keep all server side code and markup in the Razor view. Using this approach we are actually just using the UserControl SPEAK component whilst the Advanced approach shows how to make such a component.

```c#
@using Sitecore.Mvc
@using Sitecore.Web.UI.Controls.Common.UserControls
@model Sitecore.Mvc.Presentation.RenderingModel
@{
    var userControl = Html.Sitecore().Controls().GetUserControl(Model.Rendering);

    userControl.Requires.Script("speakexamples", "CustomControlSimple.js");
    userControl.Requires.Css("speakexamples", "CustomControlSimple.css");
    userControl.Class = "custom-control-simple";

    userControl.Attributes.Add("data-sc-attributename",
        userControl.GetString("RENDERINGPARAMETERNAME", "attributename", string.Empty));
    var htmlAttributes = userControl.HtmlAttributes;
}
<div @htmlAttributes >
    <!-- Implement your control markup here -->
</div>
```

First we retrieve a SPEAK component of the type UserControl using the method GetUserControl on Html.Sitecore().Controls(). _(This method is actually a ControlsExtension extension method, see the advanced approach for more info about this)_

Then we call the Requires.Script and Requires.Css to ensure that the control will render out the RequireJs attributes for these files. Note that the category written in config is used along with the name of the file to require.

Next the Class is set on the user control. This gets rendered out as the class attribute and is used for the css selector on the client side (see [part 1](/introducing-sitecore-speak-components-some-basics/ "Part 1: Introducing Sitecore SPEAK Components")).

Just as an example a data attribute is also added to the Attributes collection in the code above.

To read the parameters from the rendering in Sitecore then the UserControl component has wrapper methods for GetString, GetBool etc.

Then we render out all of the html attributes from the SPEAK usercontrol in the markup. In this example the attributes is added to an outer div which wraps the component. This could be any html element also a script tag or other which does not have any layout or presentation.

### Advanced Approach

The advanced approach is really not that advanced if you are familiar with creating custom webcontrols the good old ASP.NET way then it is actually quite straight forward.

Besides the js, cshtml and less file then also create a normal C# class.

Depending on which type of component you want to create then inherit from the corresponding base SPEAK component or ComponentBase which is at the top of the class hierarchy.

ComponentBase is an abstract class which handles rendering of a SPEAK component. Even though it somewhat resembles a WebControl then it is not, the life cycle is much different and has nothing to do with an ASP.NET WebForms Control.

It has a PreRender and a Render method though, but it is not a Control and will not be placed in a control tree the same way as ASP.NET WebForms does. The Render method is called to render the instantiated control which then ensures that PreRender is called when base.Render() is called.

Render actually just returns a string of html which is generated by an overload of Render which takes an HtmlTextWriter as a parameter.

_From reflector_

```c#
    public virtual string Render()
    {
      this.PreRender();
      this.InternalPreRender();
      StringWriter stringWriter = new StringWriter();
      this.Render(new HtmlTextWriter((TextWriter) stringWriter));
      return stringWriter.ToString();
    }
```

Then in your custom control you will just override Render(HtmlTextWriter output)

ComponentBase has two constructors which needs to be implemented as well, one parameterless and one which takes a RenderingParametersResolver as an argument. The RenderingParametersResolver can resolve parameters from the rendering in Sitecore (see [part 2](/creating-custom-sitecore-speak-components/ "Part 2: Introducing Sitecore SPEAK Components")) or if parameters is passed to the constructor from code.

_From reflector_

```c#
    protected ComponentBase()
    {
    }

    protected ComponentBase(RenderingParametersResolver parametersResolver)
    {
      this.ParametersResolver = parametersResolver;
      parametersResolver.SetComponent(this);
      this.SetProperties(parametersResolver);
    }
```

In your custom component you will need to override these and read the expected parameters from the parameter resolver.

```c#
    public CustomControlAdvanced()
    {
      InitializeProperties();
    }

    public CustomControlAdvanced(RenderingParametersResolver parametersResolver)
      : base(parametersResolver)
    {
      SomeAttribute = parametersResolver.GetString("SomeRenderingParamater", "someAttribute", string.Empty);
      InitializeProperties();
    }
```

Now to set the parameter we read from the RenderinParametersResolver as a data attribute on the control you will need to do this in the PreRender method.

```c#
    protected override void PreRender()
    {
      SetAttribute("data-sc-someattribute", SomeAttribute);
      base.PreRender();
    }
```

The ComponentBase has two important properties that has to be set. This is the DataBind value which is rendered out as the data-bind Knockout property on the html element and then there is the Class property which is the class attribute on the html element. The class is the one which is used as selector in Javascript (see [part 1](/introducing-sitecore-speak-components-some-basics/ "Part 1: Introducing Sitecore SPEAK Components")). To avoid redundant code create an InitializeProperties method and call it from both constructors.

```c#
    private void InitializeProperties()
    {
      DataBind = "MessageId: messageId";
      Class = "custom-control-advanced";
      Requires.Script("speakexamples", "CustomControlAdvanced.js");
      Requires.Css("speakexamples", "CustomControlAdvanced.css");
    }
```

The Requires call are the same as we saw in the simple approach.

A full implementation of a custom component could look like this:

```c#
namespace SpeakExamples.Controls.CustomControlAdvanced
{
  public class CustomControlAdvanced : ControlBase
  {
    public CustomControlAdvanced()
    {
      InitializeProperties();
    }

    public CustomControlAdvanced(RenderingParametersResolver parametersResolver)
      : base(parametersResolver)
    {
      SomeAttribute = parametersResolver.GetString("SomeRenderingParamater", "someAttribute", string.Empty);
      InitializeProperties();
    }

    [CanBeNull]
    protected string SomeAttribute { get; set; }

    protected override void PreRender()
    {
      SetAttribute("data-sc-someattribute", SomeAttribute);
      base.PreRender();
    }

    protected override void Render(HtmlTextWriter output)
    {
      AddAttributes(output);
      output.RenderBeginTag(HtmlTextWriterTag.Div);
      output.RenderEndTag();
    }

    private void InitializeProperties()
    {
      DataBind = "MessageId: messageId";
      Class = "custom-control-advanced";
      Requires.Script("speakexamples", "CustomControlAdvanced.js");
      Requires.Css("speakexamples", "CustomControlAdvanced.css");
    }
  }
}
```

Now when the component code is done we need to get it rendered out in a view rendering. To do this we create a class called ControlsExtension within the same namespace as the control.

```c#
namespace SpeakExamples.Controls.CustomControlAdvanced
{
  public static class ControlsExtension
  {
    [NotNull]
    public static HtmlString CustomControlAdvanced([NotNull] this Sitecore.Mvc.Controls controls, [NotNull] Rendering rendering)
    {
      Assert.ArgumentNotNull(controls, "controls");
      Assert.ArgumentNotNull(rendering, "rendering");
      var resolver = controls.GetParametersResolver(rendering);
      var control = new CustomControlAdvanced(resolver);
      return new HtmlString(control.Render());
    }

    [NotNull]
    public static HtmlString CustomControlAdvanced([NotNull] this Sitecore.Mvc.Controls controls, [NotNull] string controlId,
                                                   [CanBeNull] object parameters)
    {
      Assert.ArgumentNotNull(controls, "controls");
      Assert.ArgumentNotNull(controlId, "controlId");
      var resolver = controls.GetParametersResolver(parameters);

      var control = new CustomControlAdvanced(resolver)
        {
          ControlId = controlId
        };

      return new HtmlString(control.Render());
    }
  }
}
```

To use the extension method from the Razor view  simply call the extension method on the controls collection. The same way as we saw the GetUserControl in the simple approach.

```c#
@using Sitecore.Mvc
@using SpeakExamples.Controls.CustomControlAdvanced
@model Sitecore.Mvc.Presentation.RenderingModel
@Html.Sitecore().Controls().CustomControlAdvanced(Model.Rendering)
```

The component can then also be rendered somewhere in another rendering or even wrapped in another control. Then the ControlsExtension method can be used to pass in parameters to the component.

```c#
<div>
    @Html.Sitecore().Controls().CustomControlAdvanced("CustomControl", new { SomeAttribute = "Some Value"})
</div>
```

This is actually really clever, the same controls can get their parameters passed either from code or from Sitecore.

What approach should you choose then?

Good question. It depends on how re-usable the control should be and if it should be inserted on the page through presentation details or from a razor view.

Writing markup in code as it is done in the advanced approach in the Render method is really clumpsy and heightens maintenance time on the code. A Razor view is so much easier and clean to modify.

So it is a weigh off between the ease of these two factors which should help you decide what to do. If you are not yet familiar with using MVC in Sitecore it is easy to rely on the advanced approach, that feels so much more common and safe. But this is really just an illusion.

I could write a whole lot more about the component hierarchy in SPEAK and the parameter resolving but for now this was it for this part in the SPEAK components series.

I hope this description will help someone, it is actually extremely easy when you first get started and it really has an enormous potential.

The next part will contain a full working example of a SPEAK component using the simple approach and then I will go on writing about the SPEAK client side pipelines.
