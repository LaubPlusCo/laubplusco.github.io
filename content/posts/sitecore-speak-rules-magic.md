---
author: alc
category:
  - sitecore
  - sitecore-7.1
  - speak
  - speak-rules
date: "2013-12-22T19:47:50+00:00"
guid: http://laubplusco.net/?p=403
title: Sitecore SPEAK rules magic
url: /sitecore-speak-rules-magic/

---
At this time of year everyone is speaking about Christmas magic, but today I will show you some SPEAK magic using rules.

Following the example I gave in my last [post](/part-4-introducing-sitecore-speak-components/ "Part 4: Introducing Sitecore SPEAK Components") where we made a text clickable and then changed the background color depending on the click, now let us make a rule which hides the control when count is equal to let say 22.

## Setting up the Sitecore SPEAK rule

First we create a folder called Rules beneath the StartPage PageSettings item and in this folder we create a RuleDefinition item which we call HideControlRule.

[![Speak-rules-definition](/wp-content/uploads/2013/12/Speak-rules-definition.png)](/wp-content/uploads/2013/12/Speak-rules-definition.png)

Now we want to create a condition like this one:

[![rule definition](/wp-content/uploads/2013/12/rule_definition.png)](/wp-content/uploads/2013/12/rule_definition.png)

This makes my Sitecore Rocks throw an exception because the equals to operator item does not exist in the core database.

[![Exception thrown by Rocks](/wp-content/uploads/2013/12/rocks_exception.png)](/wp-content/uploads/2013/12/rocks_exception.png)

Everything is still good though, I notice that Rocks has inserted the same operator ID in the raw value of the rule field as expected by a switch-case in _PropertyValue.js_

```js
    switch (operatorid) {
      case "{10537C58-1684-4CAB-B4C0-40C10907CE31}":
        return propertyValue == value;
      case "{537244C2-3A3F-4B81-A6ED-02AF494C0563}":
...
```

Now when we have created the rule then we need to insert a Rule rendering on the StartPage layout. [![rule rendering parameters](/wp-content/uploads/2013/12/rule_rendering_parameters.png)](/wp-content/uploads/2013/12/rule_rendering_parameters.png)

Here we set the control to ExampleControl, the trigger as change:count and then we point to the rule item we just created.

And then we also ensure that the ExampleControl have the ID ExampleControl:

[![remember to set the id](/wp-content/uploads/2013/12/remember_to_set_the_id.png)](/wp-content/uploads/2013/12/remember_to_set_the_id.png)

Last but not least we need to make some code which listens to change on the isVisible attribute and fires a function for toggling visibility of the view element.

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

The isVisible attribute is inherited from the ControlBase SPEAK component but there is no standard implementation which shows and hides the element. I think that is kind of strange actually but it is easy to write, just a bit redundant to have in all of your custom controls (Why is this function not standard Sitecore? Am I missing something).

When clicking the ExampleControl displayText 22 times then the control hides itself as expected.

This is again a somewhat useless example but so is the entire ExampleControl. I still hope that it serves as a good example.

A more useful rule

A good time-saving example is to enable/disable a button using a rule. For example the condition could be if the text attribute of a text box is empty then the enabled attribute of a button is false. The event trigger will then be change of the text box text attribute.

This is easy to do in code but even easier to define. The rule is also more flexible and easy to extent than the code, it is simply just adding a new action to it.

Looking at the SPEAK dialog InsertAnchorDialog under /sitecore/client/Sitecore/Common/Dialogs we can see an example of such a rule.

[![Enable Button When Has Text](/wp-content/uploads/2013/12/EnableButtonWhenHasText.png)](/wp-content/uploads/2013/12/EnableButtonWhenHasText.png)

The Rule is set on the InsertAnchorDialog layout just as we saw in the ExampleControl example.

Looking at the dialog we have a bit of an issue though (just type in <hostname>/sitecore/client/Sitecore/Common/Dialogs/InsertAnchorDialog in your browser)

\- No text in the text field button is disabled, all is good.

![Disabled Button](/wp-content/uploads/2013/12/DisabledButton.png)

\- We type in text and the button enables, great.

![Enabled Button](/wp-content/uploads/2013/12/EnabledButton.png)

\- Then we remove the text again but the button is still enabled. Not that great.

![Button Still Enabled](/wp-content/uploads/2013/12/ButtonStillEnabled.png)

#### Fixing the Sitecore standard dialog

So to fix this we need to make a "reverse" rule which states that when the Text.text attribute is equal to empty string then disable the button.

_Notice that empty string is not "" it is a real empty string as value. Rocks will write value when you open the rules editor again but it is really just an empty string in the raw value._ [![Disable Button Rule](/wp-content/uploads/2013/12/DisableButtonRule.png)](/wp-content/uploads/2013/12/DisableButtonRule.png)

Then we insert a Rule rendering on the InsertAnchorDialog and point to our new rule item, set the trigger to change:text and set the TargetControl to Text.

[![Disable Button Rule Rendering](/wp-content/uploads/2013/12/DisableButtonRuleRendering.png)](/wp-content/uploads/2013/12/DisableButtonRuleRendering.png)

And now it works both ways, the button is disabled again when the text attribute of the Text Textbox is empty.

## Dissecting the SPEAK rules engine

All conditions and actions used in SPEAK are placed as items in the core database under /sitecore/client/Speak/Layouts/Renderings/Resources/Rule/Rules

[![Actions and condition items](/wp-content/uploads/2013/12/actionsandconditions.png)](/wp-content/uploads/2013/12/actionsandconditions.png)

Each item contains the syntax used by the Action or Condition which is quite straight forward. It seems like it should be easy to create your own but I have not tried this yet.

The JavaScript for the actions and conditions is placed in the physical folder <website>\\sitecore\\client\\shell\\Speak\\Layouts\\Renderings\\Resources\\Rules\\ConditionsAndActions\

Which is set as the category rules for the SPEAK custom handler / RequireJS.

```xhtml
      <speak.client.resolveScript>
        <processor type="Sitecore.Resources.Pipelines.ResolveScript.Main, Sitecore.Speak.Client" />
        <processor type="Sitecore.Resources.Pipelines.ResolveScript.Rule, Sitecore.Speak.Client" />
        <processor type="Sitecore.Resources.Pipelines.ResolveScript.ResolveBaseComponent, Sitecore.Speak.Client" />
        <processor type="Sitecore.Resources.Pipelines.ResolveScript.Controls, Sitecore.Speak.Client">
          <sources hint="raw:AddSource">
            <source folder="/sitecore/shell/client/Speak/Assets" deep="true" category="assets" pattern="*.js" />
            <source folder="/sitecore/shell/client/Speak/Layouts/Renderings" deep="true" category="controls" pattern="*.js,*.css" />
            <source folder="/sitecore/shell/client" deep="true" category="client" pattern="*.js,*.css" />
            <source folder="/sitecore/shell/client/speak/layouts/Renderings/Resources/Rules/ConditionsAndActions" deep="true" category="rules" pattern="*.js" />
            <source folder="/Components/SpeakExamples/Controls" deep="true" category="speakexamples" pattern="*.js,*.css" />
          </sources>
        </processor>
      </speak.client.resolveScript>
```

Now from looking in the markup for the rule we just created we can see that there is a data attribute called sc-rulescript which is a path to rule.js with a query parameter that contains the ID of the rules definition.

```xhtml
<div data-sc-id="HideControl" data-sc-rulescript="/sitecore/shell/-/speak/v1/rules/rule.js?i={5B031C60-7463-4769-8478-B3431303D312}&amp;f=Rule" data-sc-trigger="change:count" data-sc-control="ExampleControl" class="sc-rule" data-sc-require="/-/speak/v1/controls/Rule.js">
</div>
```

This is an auto-generated rule Javascript which looks like this for the ExampleControl rule we created in the start of this post.

```js
define(["/-/speak/v1/rules/PropertyValue.js", "/-/speak/v1/rules/SetComponentVisibility.js", "/-/speak/v1/rules/SetComponentEnabled.js", "/-/speak/v1/rules/MakeHyperLink.js"],
  function(condition_0, condition_1, condition_2, action_0, action_1, action_2, action_3) {
    var rule = function(context) {
      var rule0 = function() {
        var condition_A34679DDD2684657B86C400C73C2C654 = function() {
          return condition_1(context, {"propertyname": "ExampleControl.count", "operatorid": "{10537C58-1684-4CAB-B4C0-40C10907CE31}", "value": "22"});
        }

        var action_3CB1DDEB8C77433E88B34229AF465769 = function() {
          action_1(context, {"name": "ExampleControl", "value": "false"});
        }

        var conditions = function() {
          return condition_A34679DDD2684657B86C400C73C2C654();
        }

        var actions = function() {
          action_3CB1DDEB8C77433E88B34229AF465769();
        }

        if (conditions()) {
          actions();
        }
      };

      rule0();
    };

    return rule;
  });
```

Extremely simple actually but also extremely powerful.

So what can Sitecore SPEAK rules be used for? In the long run editors might be able to define their own rules and then you would not need a developer for making a simple Sitecore application at all, just a well-trained editor. The UI will need quite some improvements before this could become a reality. But is it feasible at all? Not sure, and we developers like to code everything ourselves most of the time which also can be an obstacle. If creating SPEAK rules instead of coding can save us time then it will be more than welcome. To really help us save time it will require that a larger collection of actions and conditions exists out-of-the-box though. I would also like to see some more documentation from Sitecore in how to write new conditions and actions even though it seems quite easy. For example an action for calling some function on a component would be quite useful. I've implemented this action along with a condition in my [next post](/creating-custom-sitecore-speak-rule-conditions-actions/ "Creating custom Sitecore SPEAK rule Conditions and Actions").
