---
author: alc
category:
  - google-page-speed
  - modules
  - pipelines
  - sitecore
cover:
  alt: crunchpng
  image: /wp-content/uploads/2013/10/crunchpng.png
date: "2013-10-26T10:22:50+00:00"
guid: http://laubplusco.net/?p=227
title: Crush png in Sitecore
url: /crush-png-in-sitecore/

---
_This post describes a module to crush png in Sitecore which can be downloaded from [Sitecore MarketPlace](http://marketplace.sitecore.net/en/Modules/CrunchPng.aspx) and the latest version of the code is also available free on [github](https://github.com/anderslaub/SitecoreCrunchPng "crunch png in sitecore code on github")._ ![Chrush png in Sitecore](/wp-content/uploads/2013/10/CrunchPng.png)

### Continuing my fight with [Google Page Speed](/minify-html-in-sitecore-pleasing-google-page-speed/ "Minify html in Sitecore").

.NET does not offer an approach for compressing png files other than the default compression. This makes it hard to please google page speed automatically since one of their checks is if the images on a site are losslessly compressed.

To compress a png file further than the default compression and still without loss then unnecessary metadata must be removed from the file. I do not really know which metadata this is or how it is removed. Several tools have been developed by other people who specializes in image compression.

Crushing a png file can take time depending on its size so it is not something which can be done on-the-fly at least not without proper support in .NET (a small plead to Microsoft).

So to make sure that png images can be crushed by Sitecore editors I made a button for the Media tab in the content editor/media library ribbon which is shown when the selected item is a png file.

When the button is clicked a custom pipeline is started which calls the command line version of [pngcrush](http://pmt.sourceforge.net/pngcrush/ "PNG Crush"). When the png file has been crushed it is saved back in the media library either replacing the existing media item or creating a new depending on a setting.

### The code for crushing

First  I thought about which steps was required to crush the png and which arguments they should resolve. I then created a new include config fileand declared the following [custom pipeline](/creating-custom-pipeline-in-sitecore/ "Creating a custom pipeline in Sitecore"):

```xhtml
<pipelines>
  <crunchPng>
      <processor type="LaubPlusCo.CrunchPng.Pipeline.LocatePngCrunchExecutable, LaubPlusCo.CrunchPng" />
      <processor type="LaubPlusCo.CrunchPng.Pipeline.CreateTemporaryFiles, LaubPlusCo.CrunchPng" />
      <processor type="LaubPlusCo.CrunchPng.Pipeline.CrunchPng, LaubPlusCo.CrunchPng" />
      <processor type="LaubPlusCo.CrunchPng.Pipeline.SaveCrunchedPngAsMediaItem, LaubPlusCo.CrunchPng" />
  </crunchPng>
</pipelines>
```

The arguments should contain the media item to crunch, two temporary file paths for source and destination files and the path to the pngcrush executable.

```c#
public class CrunchPngPipelineArgs : PipelineArgs
{
  public CrunchPngPipelineArgs(Item mediaItem)
  {
    ImageToCrunch = new MediaItem(mediaItem);
  }

  public MediaItem ImageToCrunch { get; set; }
  public string TemporarySourceFile { get; set; }
  public string TemporaryDestinationFile { get; set; }
  public string PngCrunchExecutable { get; set; }
}
```

#### Locating the pngcrush executable

First step is to locate the pngcrush executable which is located in the /bin directory. If it does not exist then break the pipeline with an error message.

```c#
public class LocatePngCrunchExecutable
{
  public void Process(CrunchPngPipelineArgs args)
  {
    args.PngCrunchExecutable = HttpContext.Current.Server.MapPath("/bin/pngcrush_1_7_67_w64.exe");
    if (!string.IsNullOrEmpty(args.PngCrunchExecutable) && File.Exists(args.PngCrunchExecutable))
      return;
    Log.Warn("Cannot find pngcrunch executable", this);
    args.AddMessage("Cannot find pngcrunch executable", PipelineMessageType.Error);
    args.AbortPipeline();
  }
}
```

Note, a x64 bit version of pngcrush is used in the module.

#### Creating the temporary files

Next thing is to create a temporary file for the source and stream the media item content into this file.

```c#
public class CreateTemporaryFiles
{
  public void Process(CrunchPngPipelineArgs args)
  {
    CreateFiles(args);
    WriteMediaStreamToSourceFile(args);
  }

  protected virtual void CreateFiles(CrunchPngPipelineArgs args)
  {
    args.TemporarySourceFile = CreateTempPngFile();
    args.TemporaryDestinationFile = args.TemporarySourceFile.Replace(".png", "_crunched.png");
  }

  protected virtual string CreateTempPngFile()
  {
    var tempFilePath = Path.GetTempFileName();
    var tempFilePngPath = tempFilePath.Replace(".tmp", ".png");
    File.Move(tempFilePath, tempFilePngPath);
    return tempFilePngPath;
  }

  protected virtual void WriteMediaStreamToSourceFile(CrunchPngPipelineArgs args)
  {
    var mediaStream = args.ImageToCrunch.GetMediaStream();
    var fileStream = File.Open(args.TemporarySourceFile, FileMode.OpenOrCreate, FileAccess.ReadWrite);
    mediaStream.CopyTo(fileStream);
    fileStream.Flush(true);
    fileStream.Close();
  }
}
```

#### Crushing the png

Now we have the executable, the source file and the name of the destination file so all that is left is to call the pngcrush executable with these parameters.

```c#
public class CrunchPng
{
  public bool UseBrute
  {
    get { return Sitecore.Configuration.Settings.GetBoolSetting("CrunchPng.UseBrute", false); }
  }

  public void Process(CrunchPngPipelineArgs args)
  {
    var startInfo = new ProcessStartInfo
      {
        CreateNoWindow = true,
        UseShellExecute = false,
        FileName = args.PngCrunchExecutable,
        WindowStyle = ProcessWindowStyle.Hidden,
        Arguments = GetCrunchArguments(args)
      };
    try
    {
      using (var exeProcess = System.Diagnostics.Process.Start(startInfo))
      {
        exeProcess.WaitForExit();
      }
    }
    catch (Exception exception)
    {
      Log.Error("Could not call png crunch", exception, this);
      args.AddMessage("Could not call png crunch", PipelineMessageType.Error);
      args.AbortPipeline();
    }
  }

  protected virtual string GetCrunchArguments(CrunchPngPipelineArgs args)
  {
    return string.Concat(UseBrute ? " -brute " : string.Empty, args.TemporarySourceFile, " ", args.TemporaryDestinationFile);
  }
}
```

I also added a setting for using the -brute argument for pngcrush.

#### Saving the crushed png

Last step is to save the crushed png back into the media library. I made a setting for either instructing the module to replace the existing media item or to create a new one. The new one will be prefixed with the word \_crunched.

If the destination file does not exist then it is because the source file was really not a png file. This can happen since someone might have named another image type as .png and then pngcrush will fail but it does not throw an exception.

```c#
public class SaveCrunchedPngAsMediaItem
{
  public bool ReplaceExisting
  {
    get { return Sitecore.Configuration.Settings.GetBoolSetting("CrunchPng.ReplaceExisting", false); }
  }

  public void Process(CrunchPngPipelineArgs args)
  {
    if (!File.Exists(args.TemporaryDestinationFile))
    {
      args.AddMessage("File is not a real png image", PipelineMessageType.Error);
      args.AbortPipeline();
      return;
    }
    var path = args.ImageToCrunch.InnerItem.Paths.FullPath;
    using (new SecurityDisabler())
    {
      var options = new MediaCreatorOptions
        {
          AlternateText = args.ImageToCrunch.Alt,
          Database =  args.ImageToCrunch.Database,
          Destination = ReplaceExisting ? path : string.Concat(path, "_crunched"),
          FileBased =  args.ImageToCrunch.FileBased,
          Language =  args.ImageToCrunch.InnerItem.Language,
          IncludeExtensionInItemName =  false,
          KeepExisting = false
        };
      MediaManager.Creator.CreateFromFile(args.TemporaryDestinationFile, options);
    }
  }
}
```

#### Inserting the button

Now the pipeline is ready so all that is left is to start the pipeline from a command which then can be mapped to a new button in the media tab.

```c#
public class CrunchPngCommand : Command
{
  public override void Execute(CommandContext context)
  {
    if (!context.Items.Any())
      return;
    Assert.IsNotNull(context.Items[0], "Error, no item in context");
    var pipelineArgs = new CrunchPngPipelineArgs(context.Items[0]);
    CorePipeline.Run("crunchPng", pipelineArgs);
    if (!pipelineArgs.Aborted && string.IsNullOrEmpty(pipelineArgs.Message))
      return;
    var errorMessage = pipelineArgs.GetMessages().FirstOrDefault(m => m.Type == PipelineMessageType.Error);
    SheerResponse.Alert(errorMessage != null ? errorMessage.Text : "An error occurred", true);
  }

  public override CommandState QueryState(CommandContext context)
  {
    Error.AssertObject(context, "context");
    if (context.Items.Length == 0)
      return CommandState.Hidden;
    if (!IsPngMediaItem(context.Items[0]))
      return CommandState.Hidden;
    return base.QueryState(context);
  }

  protected bool IsPngMediaItem(Item item)
  {
    var mediaItem = new MediaItem(item);
    return mediaItem.Extension.Equals("png", StringComparison.InvariantCultureIgnoreCase)
            || mediaItem.MimeType.Equals("image/png", StringComparison.InvariantCultureIgnoreCase);
  }
}
```

The command hides the button if the currently selected item is not a png file.

In Sitecore we create a new button under /sitecore/content/Applications/Content Editor/Ribbons/Contextual Ribbons/Images/Media/Image in the core database.

[![buttoncrushpnginsitecore](/wp-content/uploads/2013/10/buttoncrushpnginsitecore.png)](/wp-content/uploads/2013/10/buttoncrushpnginsitecore.png)

To map the button with the command we make the following entry in /App\_Config/Commands.config

```xhtml
<command name="media:pngcrunch" type="LaubPlusCo.CrunchPng.InfraStructure.CrunchPngCommand, LaubPlusCo.CrunchPng" />
```

Now when we select a png media item in the media library the button will appear.

[![CrunchPng](/wp-content/uploads/2013/10/CrunchPng.png)](/wp-content/uploads/2013/10/CrunchPng.png)

In this version there is no progress indication when the button is pressed and it can take even up to some minutes if the png is a MB or more. It works very fast for small png files though.

By the way the extra settings for replacing existing item and using the -brute argument looks like this:

```xhtml
<settings>
  <setting name="CrunchPng.ReplaceExisting" value="true" />
  <setting name="CrunchPng.UseBrute" value="false" />
</settings>
```

Final question, why is the module called CrunchPng and not CrushPng. Well to be honest mostly due to a small distraction at first and then not willing to rename the whole project after I got started. I think it makes sense though since the module crunches the data of the png file through to crush its file size. And no I am not renaming the module.

That was it, quite straight forward.
