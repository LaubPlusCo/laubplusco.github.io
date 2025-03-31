---
author: ap
category:
  - lucene
  - sitecore-6.6
  - tips
date: "2013-11-29T10:46:07+00:00"
guid: http://laubplusco.net/?p=287
title: Sitecore 7, Lucene 3.0 and Highlighted results
url: /sitecore-7-lucen-3-0-highlighted-results/

---
For all you that will use Lucene 3.0 under Sitecore 7 and want to highlight your results using Luce.Net.Contrib libraries, let me save you a whole lot of time.

On Lucene.net page [http://lucenenet.apache.org/](http://lucenenet.apache.org/) you will find a reference to the above mentioned package which i used for retrieving highlighted results from my Lucene index. All nice and smooth until i performed the search and stuck with this error for 3 days:

_Method not found: 'System.Collections.Generic.ISet\`1<!!0> Lucene.Net.Support.Compatibility.SetFactory.CreateHashSet()'._

Problem is that the out of the box Lucene.Net.dll that comes with Sitecore is not compatible with the Contrib libraries. So that dll has to be replaced with the one found in the Lucene.Net NuGet package.
