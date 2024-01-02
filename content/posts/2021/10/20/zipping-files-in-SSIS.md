---
title: "Zipping Files in SSIS"
date: 2021-10-20T12:51:10-04:00
draft: false

thumbnail: images/zip_file.jpg

tags: 
  - "ssis"

socialshare: true
---

### ***It's possible!*** ###

I've been looking for the best way to zip files using SSIS for a while now. Every time I search, however, the recommended solution has always been to use the Execute Process task and call 7-Zip (or your favorite zip utility) either directly or using a .bat file. This has been working just fine so far, but I've been on a bit of a mission to remove as many unnecessary third-party dependencies from the server as I can. 

I'm very happy to report that I've found a way to zip files using the Script Task, and even better, it's incredibly easy to implement.

Add a Script task to your package and double-click to open the task editor. In my example, I have a variable that I'm using for the path of the exported file. Make sure to add that variable to the list of read-only variables. 

![](/images/ZipFiles_ScriptTaskConfig.png)

Then click "Edit Script..." to launch into the code. The first thing we need to do is add a reference to ``System.IO.Compression`` and ``System.IO.Compression.FileSystem``. So, in Solution Explorer, right-click on References and click Add Reference. In the Reference Manager window, you can search for “System.IO.Compression”. Select those references and click OK.

![](/images/ZipFiles_ReferenceManager.png)

At the top of the code file, expand the ``Namespaces`` region and add the following:
```C#
using System.IO;
using System.IO.Compression;
```

Then, go down into ``Main()`` and add your code. 

```C#
string filePath = (string)Dts.Variables["User::FullExportPath"].Value;

FileInfo fileInfo = new FileInfo(filePath);
string zipPath = filePath.Replace(fileInfo.Extension, ".zip");

CompressFile(filePath, zipPath);            

Dts.TaskResult = (int)ScriptResults.Success;
```

So what are we doing here? Firstly, we are reading the variable for the path of the file we want to zip up. Next, I'm creating a filename of the zip file we want to create. In this example, I'm just using the same filename with a .zip extension in the same folder as the original file. If you are familiar with 7-Zip and the Windows shell extension, this is a default behavior. (Right-click -> 7-Zip -> Add to <<filename>>.zip) In truth, it's probably a good idea to have a second variable for the zip file path as well and pass that in along with the path to the original file. 

Next, I'm calling a function that takes the original file path and the path of the desired zip file as arguments. The definition is below:

```C#
private static void CompressFile(string filePath, string zipFilePath)
{
    FileInfo fileInfo = new FileInfo(filePath);
    using (FileStream zipFileStream = new FileStream(zipFilePath, FileMode.Create))
    using (ZipArchive zipFile = new ZipArchive(zipFileStream, ZipArchiveMode.Create, false))
    {
        zipFile.CreateEntryFromFile(filePath, fileInfo.Name, CompressionLevel.Optimal);
    }
}
```

Here, we are creating a ``FileStream`` object so we can write data to a file. Then we create a ``ZipArchive`` object that uses our ``FileStream`` to actually work with it as a zip file. Then, we create an entry in the zip file using the original file. Pass in the name you want to call it in the zip archive and the compression level and you are done.

Save the code file and close the VSTA editor. Click OK in the task editor window. 

Unzipping is easy as well. To unzip all the files in a zip file to a directory, you can call:

```C#
zipFile.ExtractToDirectory(destinationDirectoryName);
```

Now you can zip and unzip files in your SSIS packages without relying on third-party applications being installed in all your environments. This all falls apart, however, if your zip files have or require a password. ``System.IO.Compression`` is focused only on that - compression. To work with password protected zip files, it appears that you would still need to use a third-party solution, unfortunately.