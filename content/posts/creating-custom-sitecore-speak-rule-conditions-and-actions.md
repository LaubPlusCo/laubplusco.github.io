---
author: alc
category:
  - sitecore
  - sitecore-7.1
  - speak
  - speak-rules
cover:
  alt: speakexample
  image: /wp-content/uploads/2013/12/speakexample.png
date: "2013-12-22T23:25:24+00:00"
guid: http://laubplusco.net/?p=461
title: Creating custom Sitecore SPEAK rule Conditions and Actions
url: /creating-custom-sitecore-speak-rule-conditions-actions/

---
After writing my previous post about [Sitecore SPEAK rules magic](/sitecore-speak-rules-magic/ "Sitecore SPEAK rules magic") I got really excited when I realized how extremely easy it is to add new actions and conditions to the SPEAK rules engine. So here goes.

To make any real sense of this post I would suggest that you start reading from [my first post about SPEAK](/introducing-sitecore-speak-components-some-basics/ "Part 1: Introducing Sitecore SPEAK Components") if you haven't read the previous posts already.

## Setting up the environment

First we create some new folders in our SpeakExamples project from the previous posts. One folder called Rules with two sub-folders, one called Actions and one called Conditions.

[![rules and action folders](/wp-content/uploads/2013/12/rulesandactionfolders.png)](/wp-content/uploads/2013/12/rulesandactionfolders.png)

Then we add a new source element for the speak.client.resolveScript pipeline in Sitecore.Speak.Config

```xhtml
        <processor type="Sitecore.Resources.Pipelines.ResolveScript.Controls, Sitecore.Speak.Client">
          <sources hint="raw:AddSource">
            <source folder="/sitecore/shell/client/Speak/Assets" deep="true" category="assets" pattern="*.js" />
            <source folder="/sitecore/shell/client/Speak/Layouts/Renderings" deep="true" category="controls" pattern="*.js,*.css" />
            <source folder="/sitecore/shell/client" deep="true" category="client" pattern="*.js,*.css" />
            <source folder="/sitecore/shell/client/speak/layouts/Renderings/Resources/Rules/ConditionsAndActions" deep="true" category="rules" pattern="*.js" />
            <source folder="/Components/SpeakExamples/Controls" deep="true" category="speakexamples" pattern="*.js,*.css" />
            <source folder="/Components/SpeakExamples/Rules" deep="true" category="rules" pattern="*.js" />
          </sources>
        </processor>
```

In this new element we "extend" the category named _rules_ which usually only cover the standard rules folder to also include our new Rules folder.

This allows us to add new JavaScript in this category without having to store our code under the /sitecore folder.

You could also create the JavaScript files directly in the Sitecore folder \\sitecore\\shell\\client\\Speak\\Layouts\\Renderings\\Resources\\Rules\\ConditionsAndActions\\and skip this step but I think it is really bad practice to put your own code in the \\sitecore folder except if there is no way around it which there is in this case.

## Creating the Condition

Under the item  /sitecore/client/Speak/Layouts/Renderings/Resources/Rule/Rules/ we create a new _Condition_ item which we call _ModuloValueEqualToZero_ and set the text field to:

```amigados
where the component [name,,,name] attribute [attribute,,,attribute] modulo [value,,,value] is equal to zero
```

 [![Condition Modulo Value Equal to Zero](/wp-content/uploads/2013/12/ConditionModuloValueEqualZero.png)](/wp-content/uploads/2013/12/ConditionModuloValueEqualZero.png)

Then we create a JavaScript file beneath the newly created project folder /Rules/Conditions which we call the same name as the item _ModuloValueEqualToZero.js_ this file returns a condition.

The name of the JavaScript file has to be the exact same name as the Rules item, this is what is used to look up the rules implementation. Not sure I like that it is based on the item name but it is easy to work with.

```js
define([], function () {
  var condition = function (context, args) {
    var componentName = args.name;
    var attribute = args.attribute;
    var value = args.value || 0;

    var component = context.app[componentName];

    if (component == null) {
      throw "Component '" + componentName + "' not found";
    }

    var attributeValue = component.get(attribute);

	if (attributeValue == null) {
      throw "Attribute " + attribute " does not exist on '" + componentName;
    }

	if (typeof attributeValue !== 'number')
		attributeValue = parseInt(attributeValue);

    return attributeValue % value == 0;
  };

  return condition;
});
```

The code should be rather self-explanatory. First we read the parameters which is passed from the rule, notice that the parameter names in the args object is the names we set in the text field on the rules item.

Then we validate the parameters and finally check if the attribute value modulo the passed value is equal to zero.

## Creating the Action

Under /sitecore/client/Speak/Layouts/Renderings/Resources/Rule/Rules/Actions/ we create a new Action item called CallFuntion and set the text field to:

```amigados
call function [function,,,function] on component [name,,,name]
```

 [![CallFunction Action](/wp-content/uploads/2013/12/CallFunctionAction.png)](/wp-content/uploads/2013/12/CallFunctionAction.png)

Then as we did when we created the condition, we create a JavaScript file for the Action item which we call CallFuntion.js in the folder /Rules/Actions in our SpeakExamples project.

```js
define([], function () {
  var action = function (context, args) {
    var functionName = args.function;
    var componentName = args.name;
    var component = context.app[componentName];

    if (component == null) {
      throw "Component " + component + " does not exist";
    }

    var fn = component.viewModel[functionName];

    if (typeof fn !== "function") {
      throw functionName + " is not a function on " + component;
    }

    fn(component);
  };

  return action;
});
```

 _Notice that we pass the component as a parameter to the function. This will make the function handle "this" as the component so we do not have to re-write the function to be out of context. This action will only work for calling parameter-less functions but that can easily be modified if desired._

This code should be rather self-explanatory as the Condition code was but this on the other hand really shows the power of JavaScript. By using reflection in C# the same could be achieved as well but with a whole lot more pitfalls, this just runs without complaints.

I really like this Action, I think it should be part of the standard API. Even though it is a bit like having a call to eval(someString) from a rule just a bit more clean.

## Setting up the rule

 _First of all make sure everything is saved and then also recycle the app pool of your site or do an iisreset. Otherwise Require will not find your new JavaScript files. These are cached on initialize._

Now we go back to the SpeakExamples StartPage item and add a new _RuleDefinition_ item beneath the Rules folder which we call CountChangedRule:

[![Count Changed Rule](/wp-content/uploads/2013/12/CountChangedRule.png)](/wp-content/uploads/2013/12/CountChangedRule.png)

Then we edit the rule to use our new custom condition and action as follows:

[![The SPEAK Rule using our new condition and action](/wp-content/uploads/2013/12/TheSPEAKRule.png)](/wp-content/uploads/2013/12/TheSPEAKRule.png)

Now we do not need to listen for the change of count in the ExampleControl.js so we remove that one line and we can also remove the if statement checking if count modulo 2 is equals to zero (no lines of code saved really, but we gained a whole lot of flexibility).

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
//      this.model.on("change:count", this.onCountChanged, this);
      this.model.on("change:isVisible", this.toggleVisible, this);
    },
    toggleVisible: function () {
      if (!this.model.get("isVisible")) {
        this.$el.hide();
      } else {
        this.$el.show();
      }
    },
    onTextClicked: function () {
      this.model.set("count", this.model.get("count") + 1);
    },
    onCountChanged: function () {
//      if (this.model.get("count") % 2 == 0) {
        this.green = !this.green;
        this.$el.css("background-color", this.green ? "lightgreen" : "red");
        this.model.set("displayedText", this.reverse(this.model.get("displayedText")));
//      }
    },
    reverse: function (text) {
      return text.split("").reverse().join("");
    }
  });
});
```

Finally we add a new Rule rendering to the StartPage of our SpeakExample application:

[![count changed rule on layout](/wp-content/uploads/2013/12/countchangedruleonlayout.png)](/wp-content/uploads/2013/12/countchangedruleonlayout.png)

Here we set the trigger to change:count and the TargetControl to ExampleControl.

Voila, and now we have the exact same useless functionality as before just made with a custom condition and action.

It is actually really easy to create conditions and actions for the SPEAK rules engine. Even easier than doing it in C#.

Merry Christmas everyone. I hope someone can use this post. Please feel free to comment, thanks.

The code can be downloaded here: [SpeakExamples.zip](/wp-content/uploads/2013/12/SpeakExamples.zip)

And the serialized items here: [serialization.zip](/wp-content/uploads/2013/12/serialization.zip)

Extract the project from SpeakExamples.zip in a new folder beneath your Sitecore installation. Extract the serialization.zip archive into a folder called serialization in /App\_data and run Tools - Serialization - Update Tree using Sitecore Rocks on the item /sitecore/client/Your Apps and /sitecore/client/Speak/Layouts/Renderings/Resources/Rule/Rules
