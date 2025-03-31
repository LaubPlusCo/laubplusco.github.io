---
author: alc
category:
  - tips
date: "2014-01-14T22:17:37+00:00"
guid: http://laubplusco.net/?p=601
title: The rather unknown StringUtil class in Sitecore
url: /rather-unknown-stringutil-class-sitecore/

---
This is just another quick tip about the Sitecore API and this one has been around forever but I rarely see it used.

I use it quite often myself when I perform URL or path manipulations so now I want to spread the knowledge so others can benefit from this helping class as well.

It is a class called _StringUtil_ which, as the name implies, contain various methods that does something useful with strings.

It contain a lot of methods and some are more or less useful but there are some which deserves to be highlighted:

- **EnsurePrefix(char prefix, string value)** \- ensures that the passed prefix parameter is set as a prefix on the string value
- **EnsurePostfix(char postfix, string value)** \- same as above just for postfix
- **RemovePrefix(char prefix, string value)** \- Removes the passed prefix parameter on the string value if it exists
- **RemovePostfix(char postfix, string value)** \- same as above just for postfix

I use these four methods very often when concatenating strings for paths or urls, removing or ensuring a / whenever needed.

_Short example taken out of context_

```c#
public static UrlString AppendPathToUrl(this UrlString url, string pathToAppend)
{
  return new UrlString(string.Concat(StringUtil.EnsurePostfix('/', url.Path), StringUtil.RemovePrefix('/', pathToAppend)));
}
```

 _Other useful methods in the class:_

- **RemoveLineFeeds(string text)** \- remove linebreaks from the string text, easier to read than _someString.Replace("\\r", "").Replace("\\n", "")_.
- **GetLastPart(string text, char delimiter, string defaultValue)** \- returns the last part of the string text which is delimited using the char delimiter. Returns default value if the delimiter is not found. _Useful for parsing placeholder names from delta renderings._

There are quite a lot of other methods to look at in the class but I never really used them, some of them are similar or identical to methods which are now part of the .NET framework. This class is old. It dates back to [at least Sitecore 5.0](http://sdn.sitecore.net/doc/api%205.0/Sitecore.StringUtilMembers.html) where it is missing some methods but most of them are there.

I really get a bad vibe from the name StringUtil. No class should ever be called Util. The name just shows that whoever wrote the class didn't really know the responsibility that the class would have in the end and was too lazy to finish the word utilities _(laziness can otherwise be a strong quality in a developer)_.

Classes named like this typically ends up breaking the [Single Responsibility Principle](http://en.wikipedia.org/wiki/Single_responsibility_principle) or are just completely tech focused which in the end leads to unmanageable spaghetti. In this case it is just fine though, it is legacy code which is still very useful.

Retrospectively I made a ton of Utils classes as well some 4-6 years ago so I am definitely not blaming or bashing anyone.

Back then it was common and you will often find classes like this in other API's as well. For example in the Android API there is a class called TextUtils which is somewhat similar in that it contains common methods used on strings. If the methods was written today in C# they would ideally be written as extension methods on a string.

A couple of other bloggers have already written posts about this class before some time ago, for example see this [nice post from Mikkel Holck Madsen](http://www.mikkelhm.dk/post/2012/05/23/Tip-of-the-week-Week-21-SitecoreStringUtil.aspx) I just found.
