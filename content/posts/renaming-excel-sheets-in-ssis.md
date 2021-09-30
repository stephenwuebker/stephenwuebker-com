---
title: "Renaming Excel Sheets In SSIS"
date: "2020-01-23"
categories: 
  - "technology"
tags: 
  - "excel"
  - "ssis"
---

One problem I face on a regular basis is when a client provides data in Excel, but the sheet is named something different every time. This creates an obvious bottleneck in ETL processes and makes any kind of automation a nightmare. What's the solution? Tell the client to stop being so stupid!

Wait.

No. Don't do that. There has to be a way to deal with this in SSIS somehow, right?

There is! SSIS provides the ability to handle this problem quite easily via scripting.

As you can see, I have an Excel file with the sheets named with a date. So the next time I receive this file, the sheet will have a different name.

![](/images/ExcelSheetBeforeRename.png)

Before I import this data, I need to rename that sheet, because the Excel Source in SSIS only lets me specify sheets by name, and that name is different each time this file comes in. If I don't rename the sheet, I would need to update sheet name in my Excel Source. Every. Single. Time.

To solve this, in my SSIS package, I'll add a script task.

![](/images/ScriptTask.png)

Double-click on the script task, and make sure the script language is set to C#. You could use Visual Basic here as well if you are more comfortable with it, but I'm using C#. Also, I have the path for the Excel file as a variable in my package. So, I want add it to the list of read-only variables the script has access to. We aren't changing the path here, so it doesn't need to be read-write.

![](/images/ScriptTask_Config-1.png)

Then, click "Edit Script..." to launch into the code. The first thing we need to do is add a reference to the Excel Interop libraries. So, in Solution Explorer, right-click on References and click Add Reference. In the Reference Manager window, you can search for "Microsoft.Office.Interop.Excel". Choose the correct version for your environment, and click OK.

![](/images/ReferenceManager.png)

At the top of the script code, expand the Namespaces region and add the following:

```c#
using Microsoft.Office.Interop.Excel;
using System.Collections.Generic;
```

Then, go down into Main() and add your script code. Basically what we are doing is opening the Excel file, looping through all the sheets and renaming them, then saving and closing out of the Excel file. I'm naming the sheets "Sheet1", "Sheet2", "Sheet3", and so on, but obviously you can name them whatever you want.

```c#
Microsoft.Office.Interop.Excel.Application app = new Microsoft.Office.Interop.Excel.Application();
Workbook xlBook;

try
{
  // Using the Excel file path variable
  xlBook = app.Workbooks.Open(Dts.Variables\["User::ExcelPath"\].Value.ToString());
  int numSheet = 1;

  // Get a list of all the sheets in the file
  // Loop through and set the names
  // I'm setting the names to Sheet1,Sheet2,Sheet3,etc.
  List<Worksheet> sheets = new List<Worksheet>();
  foreach(Worksheet xlSheet in xlBook.Worksheets)
  {
    xlSheet.Name = "Sheet" + numSheet.ToString();
    numSheet++;
  }

  xlBook.Save();
  xlBook.Close();
}
catch (Exception e)
{
  MessageBox.Show("Rename Failed: " + e.Message);
}
finally
{
  app.Quit();
  app = null;
}

Dts.TaskResult = (int)ScriptResults.Success;
```

Save the file and close the Visual Studio instance. Click OK to save the script task. Now, if I execute that task, you can see the sheet name has been updated.

![](/images/ExcelSheetAfterRename.png)

Now I can set my Excel Source to always import data from "Sheet1", and don't have to worry about changing the package every single time I get this data to import. One more thing to remember, however, is to set DelayValidation = True on any tasks that use the Excel file, or the package will fail.
