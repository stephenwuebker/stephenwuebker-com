---
title: "Embracing Change: Lessons from Abandoned Projects"
date: 2024-01-09
draft: false

thumbnail: https://files.stephenwuebker.com/logos/t-sql-tuesday-logo.jpg

tags: 
  - "T-SQL Tuesday"

socialshare: true
---

[![t-sql tuesday logo](https://files.stephenwuebker.com/logos/t-sql-tuesday-logo.jpg)](https://sqlreitse.com/2024/01/02/t-sql-tuesday-170-invite-learning-from-abandoned-projects/)

Thanks to [Reitse](https://sqlreitse.com/) for the [invitation](https://sqlreitse.com/2024/01/02/t-sql-tuesday-170-invite-learning-from-abandoned-projects/)!

Reitse Eskens has asked about our failures, which is just what I wanted to dwell on to start the year! 😆

He's specifically asked about our abandoned projects. While it's easy to label these as "failures," I don't think they are. Abandoned projects are a learning path. If you learned something during a project, it is certainly a success, at least on some level. I feel like there is usually a good reason for abandoning a project, however. A lot of times the needs and requirements will change in the middle of a project. Perhaps another technology you previously rejected has released an update with the killer feature it was missing, so you drop everything and head in that direction. Maybe your budget gets rug-pulled, and you need to abandon what you are doing or rescope and find a new and cheaper way to accomplish your goal.

Basically, **STUFF HAPPENS**, and a lot of times that *stuff* is out of our control. What matters most is how we respond to it. Resilience is a key attribute in the face of challenge. Learn to embrace change, and turn setbacks into valuable opportunities for growth and improvement. Knowledge and growth are gained when things change.

One project we recently had to leave behind was our migration from databases hosted on SQL Server 2016 on an Azure VM to Azure SQL Databases. Our company got acquired, and the new company was currently utilizing AWS rather than Azure. They wanted to simplify billing, and only pay AWS, rather than have a bill from Azure and a bill from AWS. So, we had to stop our current migrations and move the databases and server to an EC2 instance. I know, I know, we moved to AWS to simplify the billing… Incredible.

Rather than get upset about it, we treated it as an opportunity to make it a bit of a greenfield project and install a fresh SQL Server 2022 instance and clean up years and years of cruft. We were able to rethink backup strategies, users and logins, and quite a few other things. Since we are now fully in AWS, we use SQL Server 2022’s new ability to connect to S3 to do our backups and restores, for example.

Our ability to adapt transformed that project from a huge time-suck slog that nobody wanted to do into a chance to enhance our systems and practices. While your projects may take unexpected turns, it's your response to these changes that defines your success.
