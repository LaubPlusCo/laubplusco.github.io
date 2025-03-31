---
author: alc
category:
  - best-practices
  - security
  - sitecore
  - sitecore-basics
cover:
  alt: Sitecore admin
  image: /wp-content/uploads/2013/10/welcometositecore.png
date: "2013-10-29T13:50:00+00:00"
guid: http://laubplusco.net/?p=242
title: Disable the Sitecore admin user
url: /disable-sitecore-admin/

---
A basic Sitecore 1-0-1 security check is to see if the admin account still uses the standard password.

It is commonly seen that sites go into production with the admin / b still working.

A way to ensure this does not occur is simply to disable the admin account when building to release or other public facing configurations.

## Disable Sitecore admin user

We first create a setting for toggling if the admin user should be disabled.

```xhtml
<setting name="AdminUser.Enable" value="false" />
```

We then create a processor in the initialize pipeline as follows:

```c#
  public class DisableAdminUserProcessor
  {
    public void Process(PipelineArgs args)
    {
      var user = Membership.GetUser("sitecore\\admin");
      if (user == null)
        return;
      user.IsApproved = Sitecore.Configuration.Settings.GetBoolSetting("AdminUser.Enable", true);
      Membership.UpdateUser(user);
    }
  }
```

And patch it into the end of the initialize pipeline.

```xhtml
<processor type="[NAMESPACE].DisableAdminUserProcessor, [ASSEMBLY]" />
```

This processor reads the setting and enables/disables the admin user accordingly.

Next thing to do is to switch the setting value depending on build configuration. This is dependent on your build environment so this post cannot help you with this.

Note that this code will simply make the admin user not being able to log in when the setting is set to false. So if the admin user is used by anyone then this is not the code you are looking for.

That was it.
