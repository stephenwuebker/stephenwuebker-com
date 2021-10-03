---
title: "Bulk file renaming with PowerShell"
date: "2019-06-03"
categories: 
  - "technology"
tags: 
  - "powershell"
---

Here's a quick one-liner that I find super-useful. If your processes are anything like mine, you've got scripts and/or SSIS packages that are supposed to run at a certain time of the month or maybe a certain day of the week. Right?

Perhaps your jobs run and output files. Nothing out of the ordinary. Usually these work fine and everything is good... except for those times when "that guy" is late, and the data your jobs depend on isn't there on time. So you have to delay your jobs and when you can finally run them, your output files are incorrectly named.

For example, we have a job that creates a set of files monthly. The SSIS package names them according to the month in which they were run. This is normally not a problem. However, this month, the data I needed to create those files was late and I couldn't run my job until after the first of the next month. I could modify the SSIS package, but for a one-off problem, it's not worth it. (To say nothing about change tracking, etc.) It's far simpler to let the files get created and rename them.

```powershell
dir \*June\* | rename-item -NewName {$\_.name -replace "June","May"}
```

Easy, right?
