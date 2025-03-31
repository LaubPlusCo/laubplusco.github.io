---
author: eok
category:
  - best-practices
  - sitecore
  - sitecore-dms
date: "2013-10-13T18:15:05+00:00"
guid: http://laubplusco.net/?p=106
tag:
  - rules-engine
title: Developing Sitecore sites using DMS Personalization rules
url: /developing-sitecore-sites-using-dms-personalization-rules/

---
As a developer it can be a rough road to start developing with DMS. Where do I start? How do I build a site that is accommodating to DMS? These are tough questions. In this post I will give some simple pointers that should get you started quick and easy on the right path.

DMS has a thousand different applications and features, but the one that I feel is most useful for us developers and the easiest to start with is the personalization rules. These rules give us freedom and a great way to separate responsibilities away from our web controls.

## Using the rules as a starting point

When first starting to develop a new website where DMS is going to be used a good trick as a developer is to focus on the rules that the customer wants to govern his content. What should control how each part of a page is shown or hidden? This simple question will usually give a lot of valuable information, because the customer does not know about personalization rules or have not thought about how to use them but they always know how their content should behave. This usually translates directly into personalization rules that you as a developer can implement. That is if they’re not already in the standard pack.

## Personalization rules showing you the way

Having the rules written down or implemented is a good start but now that you have them what good do they do other than just helping the customer customize their content?

The trick is to use the rules to help you make the right choices when creating pages for the site. When creating a new page type then look at the page and the rules that you have and use them to split the page into the parts that needs to be customized.

This way ensures that the customer is happy because he can customize the page with his rules as he wants, and you have a way of making sure you do not create a page that is too granulated or one that isn’t granulated enough.

## Using the rules as domain constraints:

As developers we like to have things simple, the simpler the better. Using rules when developing we can now take most of the domain rules that we had to put into code and apply them afterwards. You might have to create more controls but the payoff is that you get to create simple encapsulated controls and then bind them together on a Page using rules. The login box that turns into a text that tells you your user name is a great example of two controls that does not need to know each other.

## Arsenal of rules, pew pew

Having a few rules is great, but having a hundred is better. You can easily clog up your solution with a thousand rules that nobody uses, but if you create each rule properly and make sure that the next ten controls that you create can use it too. You are going to have a good time, the flexibility of the rule engine is well worth the time spent writing proper rules that is going to be used over and over again.

So to sum up, ask your customer what they want to rule their content, use it as your guide line for creating pages. Think about how the rules will allow you to write simpler controls, and really spend the time that it takes to write a proper rule, they will become the backbone of your solution after a while.
