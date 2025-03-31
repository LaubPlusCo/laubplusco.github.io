---
author: alc
category:
  - pipelines
  - security
  - sitecore
  - tips
date: "2013-10-30T22:05:50+00:00"
guid: http://laubplusco.net/?p=262
title: Ensuring roles in Sitecore
url: /ensuring-roles-in-sitecore/

---
Roles are easy to create in Sitecore but sometimes you might want to ensure that some specific roles always exists.

Not a very common scenario but nonetheless it happens. Once upon a time we needed to be able to ensure that a long list of specific roles always existed on the production instance otherwise some of our code and an integration to an external system could fail.

Inspired by the _EnsureAnonymousUsers_ processor in the initialize pipeline I made the following small module. It is very similar to the code I shown in my previous post on how to [disable the admin user](/disable-sitecore-admin/ "Disable the Sitecore admin user").

First we make a new include config file, _ensuredroles.config_,  which contains a list of role names.

```xhtml
<configuration>
  <sitecore>
    <ensuredRoles>
      <role name="SOMEDOMAIN\ROLE ONE" />
      <role name="SOMEDOMAIN\ROLE TWO" />
    </ensuredRoles>
  </sitecore>
</configuration>
```

Then we implement a pipeline processor which iterates over the configured roles and creates them if they do not exist already.

```c#
public class EnsureRolesProcessor
{
  public void Process(PipelineArgs args)
  {
    var configuredRoles = Factory.GetConfigNodes("ensuredRoles/role");
    foreach (XmlNode configuredRole in configuredRoles)
    {
      if (configuredRole.Attributes == null)
        continue;
      var roleName = configuredRole.Attributes["name"] == null ? string.Empty : configuredRole.Attributes["name"].Value;
      if (string.IsNullOrEmpty(roleName) || Role.Exists(roleName))
        continue;
      Roles.CreateRole(roleName);
    }
  }
}
```

And then we patch the processor into the end of the initialize pipeline.

```xhtml
<processor type="[NAMESPACE].EnsureRolesProcessor, [ASSEMBLY]" />
```

That is it, yet another short post. The code can easily be extended to define roles-in-roles and the same principle can also be used for ensuring that specific users always exists. Note, remember to create any custom domains in Domains.config.
