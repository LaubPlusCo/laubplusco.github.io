---
author: alc
category:
  - configuration
  - security
  - sitecore
  - tips
date: "2013-10-17T23:32:21+00:00"
guid: http://laubplusco.net/?p=170
title: How to hijack Sitecore instance using only cookie information
url: /how-to-hijack-sitecore-instance-using-only-cookie-information/

---
Or how to scare any project manager, sales guy or customer into choosing to run their site on https.

This is not going to be a lesson in how to obtain cookie information sent over a network. You can find a ton of youtube videos and other resources on how to setup a tool like [cain and abel](http://en.wikipedia.org/wiki/Cain_and_Abel_(software)) to do this in minutes. This post is not really about Sitecore either, the example just shows a Sitecore site,

Well then what is this post about? Really nothing new just what you can do in any modern browser and on any .NET based cms. Not to point out .NET as being insecure. It is just a general fact that the internet is based on technologies made for something completely else than what it is used for today. One might say that it is made naive but retrospectively this is not the case.

## Hijack Sitecore in less than a minute

Step 0: Install Firebug in Firefox or a Chrome extension like [Edit this cookie](https://chrome.google.com/webstore/detail/edit-this-cookie/) in Chrome.

Step 1: Ask PM, Sales guy or customer to come over and look.

Step 2: Open up Chrome or Firefox with Firebug installed

Step 3: Log into a relevant Sitecore solution

Step 4: Open the developer pane / Firebug

Step 5: Copy the value of the cookie called .ASPXAUTH into notepad.

[![copycookieinfo](/wp-content/uploads/2013/10/copycookieinfo1.png)](/wp-content/uploads/2013/10/copycookieinfo1.png)

Step 6: Now either open an incognito browser window or for the drama effect move to a different computer.

Step 7: Open a browser and navigate to the same Sitecore solution as before.

Step 8: Create a new cookie called .ASPXAUTH and copy paste the value you saved in notepad. Set the domain on the cookie to the domain name and the cookie as a http session cookie.

![copycookieinfo](/wp-content/uploads/2013/10/copycookieinfo.png)Step 9: Go to http://www.RELEVANTSITECORESOLUTION.com/sitecore/shell

Step 10: Click around the content editor. Maybe delete an item or change the image on the front page to a photo of a small kitten.

That is it. This is how easy it is to hijack a session in Sitecore or any .NET solution or actually really any site which does not run on https.

How can cookie information be obtained by evil people then if they are not on the same machine as the editors? All packets sent via http can be read at any point on the network in which the packets passes through. It can even be read by machines which are on the same local network such as a public insecure wireless etc. Again see [Cain and abel](http://www.oxid.it/cain.html) or maybe [Wireshark](http://www.wireshark.org/) for a more house clean tool.

What can you do about it? This is easy, [simply ensure that all authentication information only gets transferred over the network encrypted](/https-in-sitecore/ "HTTPS in Sitecore"). 2048 bit encryption is preferred. Even though a PCI verification still only requires a minimum of 128 bit encryption then this is not really strong enough anymore. Security by obscurity is also an option to add more security to your Sitecore solution. Simply change the standard name of the cookie .ASPXAUTH in the web.config under the <forms authenthication section. More about these cookie settings in a later post.

How do I know about these tools and the large bunch of others that exists? Well first time I heard about Wireshark was at a lesson in software security at the technical university of denmark back in 2003ish. That is a long time ago and everyone else who have taken lessons in computer security knows about these tools.

In 2010ish [FireSheep](http://en.wikipedia.org/wiki/Firesheep) was launched and received a lot of public attention. Like this post there was nothing new or groundbreaking about FireSheep. Due to all the publicity it made Facebook quickly change their security policy to use https by default. The easy UI of FireSheep made hijacking sessions by duplicating insecure cookies something everyone can do.

How come no one changed or did something about the whole internet since back then? Simple answer. This is how the internet and the protocols on which it is based on is made. It is not naive or insecure. It is simply not made for what it is being used for. Should we stop using it then? No, why would we do something stupid like that. Just be aware of how the technology works and take the security measures needed for your situation. Paranoia rarely helped anyone.

Like or do not like that is the question.. Just push the button down there and I'll be happy.
