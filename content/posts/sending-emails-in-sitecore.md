---
author: alc
category:
  - sitecore
  - sunday
  - tips
cover:
  alt: mailserver
  image: /wp-content/uploads/2014/10/mailserver.png
date: "2014-10-26T12:24:05+00:00"
guid: http://laubplusco.net/?p=1197
title: Sending emails in Sitecore
url: /sending-emails-sitecore/

---
Sometimes you need to send an email from your Sitecore code.

I've seen a ton of solution implementations where the developer have written some custom class that sends out emails using a SMTP client, good old classic ASP.NET. I myself  even wrote code for that in Sitecore solutions some years ago.

That was before I noticed the very simple method on the Sitecore.MainUtil class called SendMail.

The SendMail does exactly what you expect it to do.

```c#
public void ExampleMethod(string from, string to, string subject, string body)
{
  var emailMessage = new MailMessage(from, to, subject, body);
  MainUtil.SendMail(emailMessage);
}
```

The method takes a System.Net.Mail.MailMessage as argument and sends this using the SMTP client that has been configured in the standard Sitecore MailServer settings shown below.

```xhtml
      <!--  MAIL SERVER
            SMTP server used for sending mails by the Sitecore server
            Is used by MainUtil.SendMail()
            Default value: ""
      -->
      <setting name="MailServer" value="" />
      <!--  MAIL SERVER USER
            If the SMTP server requires login, enter the user name in this setting
      -->
      <setting name="MailServerUserName" value="" />
      <!--  MAIL SERVER PASSWORD
            If the SMTP server requires login, enter the password in this setting
      -->
      <setting name="MailServerPassword" value="" />
      <!--  MAIL SERVER PORT
            If the SMTP server requires a custom port number, enter the value in this setting.
            The default value is: 25
      -->
      <setting name="MailServerPort" value="25" />

```

The MainUtil class contain a lot of other old and fun methods that stood the test of time. For example the methods used for encode name replacements is in this class as well. It is well-worth a look if you haven't looked at it already.

That was it for this post. I have one last Sitecore tip coming up today on [creating custom caches in Sitecore](/create-custom-cache-sitecore/).
