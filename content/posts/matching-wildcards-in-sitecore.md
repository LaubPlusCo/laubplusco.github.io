---
author: alc
category:
  - sunday
  - tips
cover:
  alt: wildcardparser
  image: /wp-content/uploads/2014/10/wildcardparser.png
date: "2014-10-26T11:54:48+00:00"
guid: http://laubplusco.net/?p=1194
title: Matching Wildcards in Sitecore
url: /matching-wildcards-sitecore/

---
While reading this [nice blog post by my colleague Uli about resolving a Sitecore SiteContext from an URI](http://reasoncodeexample.com/2014/10/24/resolving-the-sitecontext-by-url/) I noticed that he used a regex to match a hostname using \* as a wildcard

That made me remember the Sitecore class called Sitecore.Text.WildcardParser. I once wrote a blog post about this class but I never got around to publish it. So now I'll try again and keep it short.

The WildcardParser class is pretty unknown even though it is an oldie in the Sitecore API. It can be used to match a string value with an array of strings where a \* means wildcard. It is used by Sitecore to check if a SiteInfo class matches a hostname. An example usage of the class could look like this:

```c#
public static bool MatchesUriHost(Uri uri, string hostNameWithWildcards)
{
  var hostNameWithWildcardsParts  = WildCardParser.GetParts(hostNameWithWildcards);
  return WildCardParser.Matches(uri.Host, hostNameWithWildcardsParts);
}
```

The SiteInfo class itself contain a boolean method called Matches(string host) that uses the WildCardparser to check if the hostname matches the SiteInfo hostname.

That was it. I will post another quick Sitecore Sunday tip today on [sending emails using the Sitecore API](/sending-emails-sitecore/).
