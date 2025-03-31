---
author: alc
category:
  - sitecore
  - sitecore-8
cover:
  alt: sitecore_login
  image: /wp-content/uploads/2015/01/sitecore_login.png
date: "2015-01-19T22:31:58+00:00"
guid: http://laubplusco.net/?p=1332
title: Changing the image on the login screen in Sitecore 8
url: /changing-image-login-screen-sitecore-8/

---
I really like the new login screen in Sitecore 8. The old one was also getting very old, except for some small makeovers especially from Sitecore 5.3 to 6 and the addition of a few new applications it has been almost completely static for more than 8 years.

Once upon a time the login screen had the option of adding a partner logo and some additional text. I am not sure if anyone ever used this feature or if it was still working in the later versions of Sitecore 6 and 7.

Now instead of just being able to insert a small partner logo you can change the entire full screen background image by just changing a setting in configuration:

```xhtml
<!--  LOGIN BACKGROUND IMAGE URL
      Sets the background image used on the login page /sitecore/shell/default.aspx
      Default value: "//"
-->
<setting name="Login.BackgroundImageUrl" value="/my_new_login_wallpaper.png" />
```

![pentia sitecore login screen](/wp-content/uploads/2015/01/pentia_sitecore_login.png) _And wow, it looks good.. You just need the right background image._

The image you set in config is set as background image on the outer div using the CSS3 property background-size set to cover. This ensures that the image always cover the whole div without it being stretched out of proportions or cut within the visible area. It is also on the huge plus side that the Sitecore client no longer has to [provide support for Internet Explorer older than 9](https://kb.sitecore.net/articles/087164 "Sitecore browser support") allowing the usage of (some) CSS3 without caring about editor browser compatibility.

The license information text has also been hidden by default but if you miss this you can enable it again by changing the following setting:

```xhtml
<!--  LOGIN DISABLE LICENSE INFROMATION
      If true, Sitecore hides the "License Information" link on the login page.
      Default: true
-->
<setting name="Login.DisableLicenseInfo" value="false" />
```

And voila, the license information can be shown by clicking the License Options link that appears beneath the login button.

![license information sitecore 8](/wp-content/uploads/2015/01/license_information_sitecore_8.png)

I am not the first one to write about the missing license information and how to show this again, also see this [nice post by Robbert Hock](http://www.newguid.net/sitecore/2014/sitecore-8-help-wheres-license-information/) on the same subject.

This was just a really quick post from me today. The login screen also offers a new approach for password recovery that actually allows you to change the mail without changing the /sitecore/login/Default.aspx page itself. More on this coming up soon in a following post.
