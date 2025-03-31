---
author: alc
category:
  - sitecore-8
  - tips
  - webapi
date: "2015-01-24T13:53:48+00:00"
guid: http://laubplusco.net/?p=1397
title: Implementing a WebApi service using ServicesApiController in Sitecore 8
url: /implementing-webapi-service-using-servicesapicontroller-sitecore-8/

---
This will just be a quick post about implementing a WebApi controller in Sitecore 8 using the _ServicesApiController_ class.

_Perhaps this post can help some of the teams in the ongoing Sitecore Hackathon, who knows._

## The ServicesApiController class

Sitecore.Services introduce a new abstract class called ServicesApiController that derives from the System.Web.ApiController.

The class itself only purpose is to identify an ApiController as being a Sitecore _ServicesApiController_.

This identification is used by some WebApi filters that is added in the include config file _Sitecore.Services.client.config_

```xhtml
    <!-- SITECORE SERVICES WEB API FILTERS
         Specifies the list of Web API filters to load for request handling
    -->
    <api>
      <services>
        <configuration type="Sitecore.Services.Infrastructure.Configuration.ServicesConfiguration, Sitecore.Services.Infrastructure">
          <filters hint="list:AddFilter">
            <filter>Sitecore.Services.Infrastructure.Web.Http.Filters.AnonymousUserFilter, Sitecore.Services.Infrastructure</filter>
            <filter>Sitecore.Services.Infrastructure.Web.Http.Filters.SecurityPolicyAuthorisationFilter, Sitecore.Services.Infrastructure</filter>
            <filter>Sitecore.Services.Infrastructure.Web.Http.Filters.LoggingExceptionFilter, Sitecore.Services.Infrastructure</filter>
            <!--

            Decomment this section to mandate HTTPS for all Web API requests to the site

            <filter>Sitecore.Services.Infrastructure.Web.Http.Filters.RequireHttpsFilter, Sitecore.Services.Infrastructure</filter>

            -->
          </filters>
          <allowedControllers hint="list:AddController">
            <!--

            Add allowedController elements here for any controllers that should be exempt
            from the Sitecore Services Security Policy

            <allowedController>...</allowedController>

            -->
          </allowedControllers>
        </configuration>
      </services>
    </api>
```

Deriving a controller from the _ServicesApiController_ class instead of its base _System.Web.Http.ApiController_ ensures that the controller uses these filters and that the security policy which is set for the item web api and entity services also applies for your derived class.

## A simple example

As a very very basic example I will make a simple Controller that respond with some hardcoded JSON data and in [my next post I will show how to instead respond with data from a custom mongo DB collection](/working-custom-mongodb-collections-sitecore-8-using-webapi/ "Working with custom MongoDB collections in Sitecore 8 using WebApi").

```c#
public class CommentApiController : ServicesApiController
{
  [HttpGet]
  public IHttpActionResult Get(string id)
  {
    var comment = new Comment
    {
      Id = ID.NewID.ToGuid(),
      Name = "Test Person",
      Email = "mailme@testperson.dk",
      Subject = "I have a comment",
      CommentMessage = "This is my comment"
    };
    return new JsonResult<Comment>(comment, new JsonSerializerSettings(), Encoding.UTF8, this);
  }
}
```

Now when we have the controller in place we need to register a route to the controller.

We do this by creating a pipeline processor for the initialize pipeline. This is the same concept as I used in my previous post about [securing password recovery in Sitecore 8](/securing-password-recovery-experience-sitecore-8/ "Securing the password recovery experience in Sitecore 8") and the concept is also used by the ListManager and the PathAnalyzer applications in Sitecore 8.

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
    routes.MapHttpRoute("CommentApi", "sitecore/api/comments/{id}", new
    {
      controller = "CommentApi",
      action = "Get"
    });
  }
}
```

We then patch in this processor in the initialize pipeline using the following configuration include patch.

```xhtml
<?xml version="1.0" encoding="utf-8" ?>
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <pipelines>
      <initialize>
        <processor patch:after="processor[@type='Sitecore.Pipelines.Loader.EnsureAnonymousUsers, Sitecore.Kernel']"
          type="LaubPlusCo.Examples.RegisterHttpRoutes, LaubPlusCo.Examples" />
      </initialize>
    </pipelines>
  </sitecore>
</configuration>
```

Now when we try to request the the controller we get the following expected result on our local machine.

![servicesapiresult](/wp-content/uploads/2015/01/servicesapiresult.png)

But we get a 403 status code (forbidden) when requesting the controller from a remote machine.

![service_forbidden](/wp-content/uploads/2015/01/service_forbidden.png)

This is because of the default security policy for the item web api is set to the ServicesLocalOnlyPolicy policy that only allow requests from localhost through to the services. See the Sitecore.Services.client.config for a description of the different security policies that can be applied to ServicesApiControllers.

```xhtml
<!--  SITECORE SERVICES SECURITY POLICY
	Specifies the Sitecore.Services.Infrastructure.Web.Http.Security.IAuthorizePolicy derived type that will set the security policy
	for the Sitecore services.

	Policies:

	Sitecore.Services.Infrastructure.Web.Http.Security.ServicesOffPolicy, Sitecore.Services.Infrastructure
	- Policy denies access to all Entity and Item Services

	Sitecore.Services.Infrastructure.Web.Http.Security.ServicesLocalOnlyPolicy, Sitecore.Services.Infrastructure
	- Policy denies access to all Entity and Item Services from requests originating from remote clients

	Sitecore.Services.Infrastructure.Web.Http.Security.ServicesOnPolicy, Sitecore.Services.Infrastructure
	- Policy allows access to all Entity and Item Services

	Default: Sitecore.Services.Infrastructure.Web.Http.Security.ServicesLocalOnlyPolicy, Sitecore.Services.Infrastructure
-->
<setting name="Sitecore.Services.SecurityPolicy" value="Sitecore.Services.Infrastructure.Web.Http.Security.ServicesOffPolicy, Sitecore.Services.Infrastructure" />

```

## What are the benefits of ServicesApiController then?

If we had chosen to derive our Controller directly from the System.Web.Http.ApiController class then none of the filters that is set in config would apply to our Controller and it would be accessible for anyone and exceptions would not be logged in the Sitecore log.

That was it.

[In my next post I will show how to get and set data from a mongo db through a ServicesApiController.](/working-custom-mongodb-collections-sitecore-8-using-webapi/ "Working with custom MongoDB collections in Sitecore 8 using WebApi")
