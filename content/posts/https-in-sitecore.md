---
author: alc
category:
  - asp.net
  - pipelines
  - security
  - sitecore
  - tips
  - webforms-for-marketers
date: "2013-10-13T16:17:14+00:00"
guid: http://laubplusco.net/?p=68
title: HTTPS in Sitecore
url: /https-in-sitecore/

---
I would always recommend running all production site cms’ using security on the transport layer (https). Sitecore or no Sitecore, it is not safe to first send username and password unencrypted and then following having an insecure session cookie for the authentication information. This applies for both extranet users and the backend administrators.

A basic best practice is simply to ensure that all requests containing authentication information such as the .ASPXAUTH cookie used by ASP.NET, only gets transferred using the https scheme.

For more about this I will soon write a quick post showing how to hijack any Sitecore instance using only cookie information. I will not show how to obtain the cookie information just how to scare any project manager or site owner into requiring https.

![aspxauthcookie](/wp-content/uploads/2013/10/aspxauthcookie1.png)First step in securing your site is of course to obtain a SSL/TLS certificate of preferably minimum 256 bit encryption strength. To ease our world as developers Microsoft has made an IIS Express Certificate available in IIS 7.![iisexpresscertificate](/wp-content/uploads/2013/10/iisexpresscertificate.png)

This makes testing https an ease without having to create a certificate using open ssl as were required back in the days.

## Ensuring https in Sitecore

The simplest approach I found has been using a HttpRequestProcessor in the httpRequestBegin pipeline to check if the current request should use https and if it is not then change the scheme of the current url and redirect.

```c#
public class CheckTransportSecurityProcessor : HttpRequestProcessor
{
  public override void Process(HttpRequestArgs args)
  {
    if (Context.Item == null || !UseSecureSchemeService.Check() || args.Context.Request.Url.Scheme.Equals(Uri.UriSchemeHttps, StringComparison.InvariantCultureIgnoreCase))
      return;
    if (!Context.User.IsAuthenticated && !args.Context.Request.Url.OriginalString.Contains("/sitecore") && Context.Item != null && !RequiresTransportSecurityService.Check(Context.Item))
      return;
    args.Context.Response.Redirect(ChangeUriSchemeService.ChangeToSecureScheme(args.Context.Request.Url.AbsoluteUri), true);
  }
}
```

It can be quite annoying having to use https in the development environment which is why I’ve added a check on a custom Sitecore setting if https should be used at all. The first _if_ sentence checks if this setting is false and then if https is used already.

```c#
public class UseSecureSchemeService
{
  public static bool Check()
  {
    return Settings.GetBoolSetting("TransportLayerSecurity.UseSecureScheme", false);
  }
}
```

And in config:

```xhtml
<setting name="TransportLayerSecurity.UseSecureScheme" value="true" />
```

The following checks in the processor ensures that no matter what all authenticated requests and all requests for anything beneath /sitecore is redirected to https. Otherwise the RequiresTransportSecurityService is used to check whether or not this request requires https.

In this service I read the value of a checkbox field on the Context.Item. The check could also return true for any item residing beneath /members, /login or similar.

```c#
public class RequiresTransportSecurityService
{
  public static bool Check(Item item)
  {
    if (item == null || item.Fields[Constants.Fields.RequiresTransportSecurity] == null)
      return false;
    return new CheckboxField(item.Fields[Constants.Fields.RequiresTransportSecurity]).Checked;
  }
}
```

I like the checkbox better though since it makes it possible to set it as being ticked on any page without code or config changes. Having it ticked on a login page ensures that when logging in from that page no authenticated requests should risk running over http by accident but if the user navigates away from the page without logging in the site goes back to using http. Random pages which contain Forms created by the editors can also be secured without any need for code changes.

[![requirehttpscheckbox](/wp-content/uploads/2013/10/requirehttpscheckbox.png)](/wp-content/uploads/2013/10/requirehttpscheckbox.png)

The last thing for the pipeline processor to do is to perform the redirect. For changing the scheme on the current URI I made the following service class:

```c#
public class ChangeUriSchemeService
{
  public static Uri ChangeToSecureScheme(Uri uri)
  {
    return new Uri(ChangeToSecureScheme(uri.AbsoluteUri));
  }

  public static string ChangeToSecureScheme(string uri)
  {
    if (!uri.StartsWith(Uri.UriSchemeHttp, StringComparison.InvariantCultureIgnoreCase))
      return uri;
    var secureUri = uri.Substring(4);
    return string.Concat(Uri.UriSchemeHttps, secureUri);
  }
}
```

The processor should run after the layout for the page has been resolved which means that in an out-of-the-box Sitecore solution this config element should be patched after Sitecore.Pipelines.HttpRequest.LayoutResolver

```xhtml
<processor type="[NAMESPACE].CheckTransportSecurityProcessor, [ASSEMBLY]" />
```

That is it, this code will ensure https in your Sitecore slution.

A small note for css / js resources is never to include a scheme when referencing these that is a big no-no.

Instead use a relative path for css resources and for external JS libraries just use "//<path>" covering any scheme for example:

<script src=" [//ajax.googleapis.com/ajax/libs/jquery/1.8/jquery.min.js](http://ajax.googleapis.com/ajax/libs/jquery/1.8/jquery.min.js)"></script>
