---
author: alc
category:
  - sitecore
  - sitecore-8
  - tips
date: "2015-01-21T22:55:42+00:00"
guid: http://laubplusco.net/?p=1351
title: The amazing password recovery experience in Sitecore 8
url: /editor-password-recovery-experience-in-sitecore-8/

---
Along with the [new login screen in Sitecore 8](/changing-image-login-screen-sitecore-8/ "Changing the image on the login screen in Sitecore 8") there is also a new implementation of password recovery for editors.

In the old versions of Sitecore the /sitecore/login/default.aspx page redirected the user to a page called passwordrecovery.aspx. This page used an asp:PasswordRecovery asp.net standard webcontrol.

Personally I never became fond of all of these standard but rather complex asp.net webcontrols back when they were released with .NET 2.0. They allowed you to really quickly get a demo site up and running with standard functionality but changing any behavior just a tiny bit to fit your domain specific needs or even design took much more effort than just to use simple webcontrols to build up the same functionality. I only know about most of these webcontrols due to old Microsoft certifications where you were forced to answer all questions like you used them all the time even though you always chose not to because of their inflexible design.

Anyway, for some reason Sitecore has used the asp:PasswordRecovery control back in the days and it has worked quite fine, password recovery mails were sent and we have all been very happy or maybe rather haven't really cared, it worked.

![old_pw_recovery](/wp-content/uploads/2015/01/old_pw_recovery.png)

But there were no way of changing the text in the mails that were sent out with new passwords, not without modifying the passwordrecovery.aspx page itself and overriding its code behind where the texts were hardcoded.

```xhtml
<asp:PasswordRecovery ID="PasswordRecovery" runat="server" SuccessPageUrl="default.aspx"
   OnVerifyingUser="VerifyingUser" OnSendingMail="SendEmail" CssClass="password-recovery-control">
	<MailDefinition Priority="High" Subject="Sending Per Your Request" From="someone@example.com" />
	<InstructionTextStyle CssClass="titleText" />
	<SuccessTextStyle CssClass="success-text" />
	<LabelStyle CssClass="label" />
	<TextBoxStyle CssClass="textbox" />
	<FailureTextStyle CssClass="failure-text" />
	<SubmitButtonStyle CssClass="submit-button" />
</asp:PasswordRecovery>
```

This would then break each time Sitecore were upgraded and so on. I guess no one ever changed the password recovery mails due to this and no one really cared about it.

> In Sitecore 8 this has changed

To enable password recovery you need to ensure that you have defined an SMTP server in configuration and that password recovery is not disabled:

```xhtml
<!--  MAIL SERVER
  SMTP server used for sending mails by the Sitecore server
  Is used by MainUtil.SendMail()
  Default value: ""
-->
<setting name="MailServer" value="my.smtp.server.com" />

<!--  LOGIN DISABLE PASSWORD RECOVERY
      If true, Sitecore hides the "Forgot Your Password?" link on the login page
      Default: false
-->
<setting name="Login.DisablePasswordRecovery" value="false" />
```

![sitecore_8_password_recovery](/wp-content/uploads/2015/01/sitecore_8_password_recovery.png)

To change the text in the password recovery email you can now simply edit the item **/sitecore/system/Settings/Security/Password recovery/Password Recovery Email** in the core database.

![passwordrecoveryitem](/wp-content/uploads/2015/01/passwordrecoveryitem.png)

The item field names and what they are used for are quite obvious. The token %Password% is replaced with a generated password and %UserName% with the username (for some solutions it can be considered insecure to include the username along with the password).

When requesting a new password the typed in email address is now sent through a pipeline that out-of-the-box contain the following processors.

- **VerifyUsername** \- Verifies that a user exists with the provided user name and that the user has an email defined.
- **GeneratePassword** \- Resets the user password with a new random one.
- **PopulateMail** \- Populates the pipeline arguments with mail content using the item shown above.
- **SendPasswordRecoveryMail** \- Instantiates a MailMessage and sends it to the user using [MainUtil.SendMail(..)](/sending-emails-sitecore/ "Sending emails in Sitecore")

> _Finally it is here, what we all have been waiting for, it is now possible to change the password recovery behavior in Sitecore 8. Who need machine learning and Skynet now?_

Sitecore sends the mail as plain text and not as html in the standard implementation, but they actually left a small property in the pipeline arguments called HtmlEmailContent.

If you assign this property with a value then the SendMail processor will use this property instead of the plain text.

Below you see an example of a new PopulateMail processor that uses (hardcoded) html as mail content.

```c#
public class PopulateHtmlMail : PasswordRecoveryProcessor
{
  public override void Process(PasswordRecoveryArgs args)
  {
    args.SendFromDisplayName = "My Sitecore site";
    args.SendFromEmail = "donotreply@mysitecoresite.net";
    args.Subject = "Password recovery";
    args.HtmlEmailContent = GetHtmlEmailContent(args);
  }

  private string GetHtmlEmailContent(PasswordRecoveryArgs args)
  {
    var sb = new StringBuilder();
    sb.AppendLine("<html><head><title>");
    sb.AppendLine("Sitecore Password");
    sb.AppendLine("</title></head><body>");
    sb.AppendLine("<h1>Your password has been reset</h1>");
    sb.AppendLine("<p>Hi " + args.Username + ",<br/></p>");
    sb.AppendLine("<p>Your new password is <strong>" + args.Password + ",</strong></p>");
    sb.AppendLine("</body>");
    sb.AppendLine("</html>");
    return sb.ToString() ;
  }
}
```

_It is a very bad idea to hardcode html like this, **it is just an example, please do not use it**._

Then we patch the new processor into the pipeline instead of the PopulateMail processor. _Notice that the pipeline is defined within the configuration element <processors> along with the ui pipelines and not within the <pipelines> element._

```xhtml
<?xml version="1.0" encoding="utf-8" ?>
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
	<sitecore>
	  <processors>
		<passwordRecovery argsType="Sitecore.Pipelines.PasswordRecovery.PasswordRecoveryArgs">
		  <processor patch:instead="processor[@type='Sitecore.Pipelines.PasswordRecovery.PopulateMail, Sitecore.Kernel']"
			  mode="on" type="LaubPlusCo.PasswordRecovery.Infrastructure.PopulateHtmlMail, LaubPlusCo.PasswordRecovery" />
		</passwordRecovery>
	  </pipelines>
	</sitecore>
</configuration>
```

And now the password recovery mail look like this:

![html_recovery_mail](/wp-content/uploads/2015/01/html_recovery_mail.png)

Instead of this:

![default_pw_recovery_mail](/wp-content/uploads/2015/01/default_pw_recovery_mail.png)

It would be nice to have a standard ECM implementation that sends a one-time message for password recovery when ECM is installed. Please feel free to implement this and share the code :)

### Now to a much better example

You might want to provide some extra service for your editors and also tighten security so each time a password is reset the editor also receives a text message on his/hers phone.

To do this we first need to [add a custom property to the user profile](https://briancaos.wordpress.com/2014/06/12/sitecore-users-custom-profile-properties/) for phone number:

![custom_user_profile_phone](/wp-content/uploads/2015/01/custom_user_profile_phone.png)

We then need a (free or cheap) SMS service with a REST api. I found a completely random one called called [textlocal](http://www.textlocal.com/simple-developer-sms-api) that promise you 10 texts when you sign up. I never got any free credits and ended up purchasing 100 credits for 5£ which is almost double the price of some of their competitors. Their banner got me tricked and that is why I picked them. Their API is simple and really easy to use though.

Now we'll write a quick and dirty processor for the passwordRecovery pipeline that sends a text message when a user recover their password information:

```c#
public class SendTextMessage : PasswordRecoveryProcessor
{
  public override void Process(PasswordRecoveryArgs args)
  {
    Assert.ArgumentNotNull(args, "args");
    var user = User.FromName(args.Username, false);
    var phoneNumber = user.Profile.GetCustomProperty("Phone");
    if (string.IsNullOrEmpty(phoneNumber))
      return;
    if (Send(phoneNumber, "Hi " + user.Profile.FullName + ". Your Sitecore password has been reset to " + args.Password))
      return;
    Log.Warn("Sending password recovery text failed to " + phoneNumber, this);
  }

  protected bool Send(string phoneNumber, string message)
  {
    using (var webClient = new WebClient())
    {
      var postParameters = new NameValueCollection
      {
        {"username", "anders.laub@gmail.com"},
        {"hash", "SECRET_HASH"},
        {"numbers", phoneNumber},
        {"message", message},
        {"sender", "Sitecore site"}
      };
      var response = webClient.UploadValues("https://api.txtlocal.com/send/", postParameters);
      return ResultIsSuccess(Encoding.UTF8.GetString(response));
    }
  }

  private bool ResultIsSuccess(string response)
  {
    dynamic responseObject = JsonConvert.DeserializeObject(response);
    return responseObject.status == "success";
  }
}
```

And then we patch it into the configuration after the mail has been send.

```xhtml
<?xml version="1.0" encoding="utf-8" ?>
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
	<sitecore>
	  <processors>
		<passwordRecovery argsType="Sitecore.Pipelines.PasswordRecovery.PasswordRecoveryArgs">
		  <processor patch:after="processor[@type='Sitecore.Pipelines.PasswordRecovery.SendPasswordRecoveryMail, Sitecore.Kernel']"
			  mode="on" type="LaubPlusCo.PasswordRecovery.Infrastructure.SendTextMessage, LaubPlusCo.PasswordRecovery" />
		</passwordRecovery>
	  </pipelines>
	</sitecore>
</configuration>
```

When we recover our password we receive a text message on our phone as shown below:

![password recovery text message](/wp-content/uploads/2015/01/password_recovery_text_message.png)

That was it for this post.
