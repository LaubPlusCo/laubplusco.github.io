---
author: alc
category:
  - errors
  - sitecore
  - sitecore-updates
cover:
  alt: missing_method_on_sitecore_updatehelper_distorted
  image: /wp-content/uploads/2014/10/missing_method_on_sitecore_updatehelper_distorted.jpg
date: "2014-10-28T10:35:14+00:00"
guid: http://laubplusco.net/?p=1220
title: UpdateHelper SaveInstallationMessages() removed in Sitecore 7.5
url: /updatehelper-saveinstallationmessages-removed-sitecore-7-5/

---
Sometimes we use the Sitecore class called UpdateHelper to install packages on different Sitecore instances for example via a webservice.

Today I copied some code from a Sitecore 7.2 installation which did exactly this. It is a simple service class that installs a package from a path and then it logs the installation results as done by the Sitecore InstallUpdatePackage wizard.

```c#
public static string InstallUpdatePackage(String packageFile)
{
  var log = LogManager.GetLogger("LogFileAppender");
  string result;
  using (new ShutdownGuard())
  {
    var packageInstallationInfo = new PackageInstallationInfo
    {
      Action = UpgradeAction.Upgrade,
      Mode = InstallMode.Install,
      Path = packageFile
    };
    string text = null;
    List<ContingencyEntry> entries = null;
    try
    {
        entries = UpdateHelper.Install(packageInstallationInfo, log, out text);
    }
    catch (PostStepInstallerException ex)
    {
      entries = ex.Entries;
      text = ex.HistoryPath;
      throw;
    }
    finally
    {
      try
      {
        UpdateHelper.SaveInstallationMessages(entries, text);
      }
      catch (Exception ex)
      {
        log.Warn("Unable to write the installation messages", ex);
      }
    }
    result = text;
  }
  return result;
}
```

But the code would not compile when using the Sitecore.Update.dll from 7.5

[![missing_method_on_sitecore_updatehelper](/wp-content/uploads/2014/10/missing_method_on_sitecore_updatehelper.jpg)](/wp-content/uploads/2014/10/missing_method_on_sitecore_updatehelper.jpg)

This is because the method SaveInstallationMessages amongst others has been deleted from the UpdateHelper class without first being marked as deprecated or obsolete.

Instead Sitecore moved the logging functionality directly to the code behind of the InstallUpdatePackage page. This prevents us from re-using the code directly which is a bit of a shame.

My fix was simply to write to the Sitecore log if the installation fails and I guess this is good enough for now.

It would be best if such API methods were marked as deprecated before they were deleted. A small plead to the Sitecore dev team.

It would also be okay if it was just mentioned in the release history on SDN. I presume that the methods has been deleted in relation to this resolved issue:

[![sdn_note_on_updatewizard](/wp-content/uploads/2014/10/sdn_note_on_updatewizard.jpg)](/wp-content/uploads/2014/10/sdn_note_on_updatewizard.jpg)

The way that issue was resolved just created some new ones :)
