---
author: ap
category:
  - azure
  - webapi
date: "2014-09-24T16:59:21+00:00"
guid: http://laubplusco.net/?p=1055
title: Azure API Management and output cache policies
url: /azure-api-management-output-cache-policies/

---
I am writing this post because I was one day confronted with the situation when I had to move a whole bunch of HTTP Services to be called via Azure API Management. These services were serving mobile calls from both iOS and Android devices.

All nice and well but the problem I got was that some of these services should have their response cached for a period of time so that the apps would not call them every single time. The API management does indeed provide some settings to set caching but for some reason whenever the apps would call any of the cached services the "Cache-Control" header was always set to _private_ instead of _public_ as it was desired ( [here](http://www.mobify.com/blog/beginners-guide-to-http-cache-headers/) is some interesting reading about Cache-control headers ).

So in this picture you can see how I was setting the caching in the Azure API Management :

[![Cached Operation](/wp-content/uploads/2014/09/cachingOperation-300x241.jpg)](/wp-content/uploads/2014/09/cachingOperation.jpg)

This was not enough. The cache was still private and not having a max-age value until I specifically inserted in the caching policy of that specific operation.

[![cachingPolicies](/wp-content/uploads/2014/09/cachingPolicies-300x207.jpg)](/wp-content/uploads/2014/09/cachingPolicies.jpg)

Hope this post helps someone. Leave a message if it did ;).
