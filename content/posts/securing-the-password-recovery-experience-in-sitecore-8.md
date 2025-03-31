---
author: alc
category:
  - sitecore
  - sitecore-8
  - tips
date: "2015-01-23T16:06:31+00:00"
guid: http://laubplusco.net/?p=1384
title: Securing the password recovery experience in Sitecore 8
url: /securing-password-recovery-experience-sitecore-8/

---
Following my post on [password recovery in Sitecore 8](/editor-password-recovery-experience-in-sitecore-8/ "The amazing password recovery experience in Sitecore 8") fellow Sitecore MVP [Kam Figy](https://twitter.com/kamsar) pointed out how the default Sitecore implementation potentially can be used by a malicious individual to block an editor from logging in by resetting their password automatically.

This can be done simply by creating a script that request a new password for a known user name once every x minute/second. That would really annoy the victim and potentially also cause business havoc.

I can understand why Sitecore has not focused on this vulnerability due to the big differences in customer setups. The client login screen will in some cases be public available and in other cases be within a corporate network etc. If the Sitecore client is public available then password recovery can be completely turned off.

Anyway he makes a really good point so here is a quick and dirty implementation that sends out a mail with a confirm link before sending out an email with a new password.

First we do some configuration re-organizing.

We copy paste the old passwordRecovery pipeline and call the new one confirmedPasswordRecovery

```xhtml
<?xml version="1.0" encoding="utf-8" ?>
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
	<sitecore>
	  <processors>
      <confirmPasswordRecovery argsType="Sitecore.Pipelines.PasswordRecovery.PasswordRecoveryArgs">
        <processor mode="on" type="Sitecore.Pipelines.PasswordRecovery.VerifyUsername, Sitecore.Kernel" />
        <processor mode="on" type="Sitecore.Pipelines.PasswordRecovery.GeneratePassword, Sitecore.Kernel" />
        <processor mode="on" type="Sitecore.Pipelines.PasswordRecovery.PopulateMail, Sitecore.Kernel" />
        <processor mode="on" type="Sitecore.Pipelines.PasswordRecovery.SendPasswordRecoveryMail, Sitecore.Kernel" />
      </confirmPasswordRecovery>
	  </processors>
	</sitecore>
</configuration>
```

The reason for this is that we still want to execute the exact same pipeline when the user has confirmed that he requested password recovery.

Then we modify the passwordRecovery that is executed by the Sitecore login page:

```xhtml
<?xml version="1.0" encoding="utf-8" ?>
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <processors>
      <passwordRecovery argsType="Sitecore.Pipelines.PasswordRecovery.PasswordRecoveryArgs">
        <processor patch:instead="processor[@type='Sitecore.Pipelines.PasswordRecovery.GeneratePassword, Sitecore.Kernel']"
          mode="on" type="LaubPlusCo.PasswordRecovery.Infrastructure.GenerateToken, LaubPlusCo.PasswordRecovery" />
        <processor patch:instead="processor[@type='Sitecore.Pipelines.PasswordRecovery.PopulateMail, Sitecore.Kernel']"
          mode="on" type="LaubPlusCo.PasswordRecovery.Infrastructure.PopulateConfirmMail, LaubPlusCo.PasswordRecovery" />
      </passwordRecovery>
    </processors>
  </sitecore>
</configuration>

```

We replace the _GeneratePassword_ with a processor called _GenerateToken_ for generating and storing a token on the user profile:

```c#
public class GenerateToken : PasswordRecoveryProcessor
{
  public override void Process(PasswordRecoveryArgs args)
  {
    Assert.ArgumentNotNull(args, "args");
    var user = User.FromName(args.Username, true);
    if (user == null)
    {
      args.AbortPipeline();
      return;
    }
    var token = ID.NewID.ToShortID().ToString();
    StoreTokenOnUser(user, token);
    args.CustomData.Add(Constants.ConfirmTokenKey, token);
  }

  private void StoreTokenOnUser(User user, string confirmToken)
  {
    user.Profile.SetCustomProperty(Constants.ConfirmTokenKey, confirmToken);
    user.Profile.Save();
  }
}
```

To make this work you need to add a field called _PasswordToken_ on the user profile. See Brians post on doing this [setting custom properties on Sitecore user profiles](https://briancaos.wordpress.com/2014/06/12/sitecore-users-custom-profile-properties/).

Then I've placed the common key for the token in a constants struct:

```c#
internal struct Constants
{
  internal const string ConfirmTokenKey = "PasswordToken";
}
```

And then we replace _PopulateMail_ with _PopulateConfirmMail_ that populates a html email with a confirm link.

```c#
public class PopulateConfirmMail : PopulateMail
{
  public override void Process(PasswordRecoveryArgs args)
  {
    Assert.ArgumentNotNull(args, "args");
    var token = args.CustomData[Constants.ConfirmTokenKey] as string;
    if (string.IsNullOrEmpty(token))
      return;
    var confirmLink = GenerateConfirmLink(token, args.Username);
    args.SendFromDisplayName = "Sitecore website";
    args.SendFromEmail = "donotreply@mysitecoresite.net";
    args.Subject = "Confirm password recovery";
    var user = User.FromName(args.Username, false);
    args.HtmlEmailContent = GetHtmlEmailContent(user, confirmLink);
  }

  protected virtual string GenerateConfirmLink(string token, string userName)
  {
    var serverUrl = StringUtil.EnsurePostfix('/', WebUtil.GetServerUrl());
    return serverUrl + "sitecore/api/passwordrecovery/confirm/" + userName.Replace('\\', '|') + "/" + token;
  }

  protected virtual string GetHtmlEmailContent(User user, string confirmLink)
  {
    var sb = new StringBuilder();
    sb.AppendLine("<html><head><title>");
    sb.AppendLine("Sitecore password recovery");
    sb.AppendLine("</title></head><body>");
    sb.AppendLine("<h1>Please confirm</h1>");
    sb.AppendLine("<p>Hi " + user.Profile.FullName + ",<br/></p>");
    sb.AppendLine("<p>Please follow the link below to recover your password</p>");
    sb.AppendLine("<a href=\"" + confirmLink + "\">" + confirmLink + "</a>");
    sb.AppendLine("</body>");
    sb.AppendLine("</html>");
    return sb.ToString();
  }
}
```

Do not hardcode html in code like this. It is only intended as example.

Notice the confirm link that we create, this is for a ApiController with a HttpGet method that expects a username and a token.

```c#
public class ConfirmRecoveryController : ApiController
{
  [HttpGet]
  public IHttpActionResult Confirm(string userName, string token)
  {
    userName = userName.Replace('|', '\\');
    var user = Sitecore.Security.Accounts.User.FromName(userName, true);
    if (user == null || !TokenIsValid(user, token))
      return new StatusCodeResult(HttpStatusCode.Unauthorized, this);

    var passwordRecoveryArgs = new PasswordRecoveryArgs(HttpContext.Current)
    {
      Username = userName
    };
    Pipeline.Start("confirmPasswordRecovery", passwordRecoveryArgs);
    if (!passwordRecoveryArgs.Aborted)
      DeleteToken(user);

    return new StatusCodeResult(HttpStatusCode.OK, this);
  }

  private void DeleteToken(User user)
  {
    user.Profile.SetCustomProperty(Constants.ConfirmTokenKey, string.Empty);
    user.Profile.Save();
  }

  private bool TokenIsValid(User user, string token)
  {
    return !string.IsNullOrEmpty(token) && ShortID.IsShortID(token) && TokenExists(user, token);
  }

  private bool TokenExists(User user, string confirmToken)
  {
    var tokenOnProfile = user.Profile.GetCustomProperty(Constants.ConfirmTokenKey);
    return !string.IsNullOrEmpty(tokenOnProfile) && tokenOnProfile.Equals(confirmToken, StringComparison.InvariantCultureIgnoreCase);
  }
}
```

This controller executes the new (old) confirmPasswordRecovery pipeline, then deletes the token and redirects to the login page.

We need to register the route so we create a processor for the initialize pipeline:

```c#
public class RegisterHttpRoutes
{
  public void Process(PipelineArgs args)
  {
    GlobalConfiguration.Configure(Configure);
  }

  protected void Configure(HttpConfiguration configuration)
  {
    var routes = configuration.Routes;
    routes.MapHttpRoute("PasswordRecovery_Confirm", "sitecore/api/passwordrecovery/{action}/{userName}/{token}", new
    {
      controller = "ConfirmRecovery",
      action = "Index"
    });
  }
}
```

And patch it into the initialize pipeline:

```xhtml
<?xml version="1.0" encoding="utf-8" ?>
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <pipelines>
      <initialize>
        <processor patch:after="processor[@type='Sitecore.Pipelines.Loader.EnsureAnonymousUsers, Sitecore.Kernel']"
          type="LaubPlusCo.PasswordRecovery.Infrastructure.RegisterHttpRoutes, LaubPlusCo.PasswordRecovery" />
      </initialize>
    </pipelines>
  </sitecore>
</configuration>

```

And voila, the new confirm email looks like this:

![confirm_mail](/wp-content/uploads/2015/01/confirm_mail.png)

When the link is clicked a new email is sent with a new password for the user.

Right now it simply returns a 200 and no redirect to the login screen or anything. This would not really be ideal without showing a message. One might argue that a better solution would be to make a new reset password page that verifies the token and the username combo and then allow the user to type in a new password. But this is not how I made it, I prefer to do it all in pipelines, no new .aspx files.

And the password recovery mail is the standard one that you can modify in the core database on the item **/sitecore/system/Settings/Security/Password recovery/Password Recovery Email**

That should be it. Please refactor the code before using it in a production environment.
