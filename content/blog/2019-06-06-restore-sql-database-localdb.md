---
title: "How to restore a SQL Azure database locally"
description:
  This guide will show you how you can create a backup of a SQL Azure Database and restore it to a LocalDB database
tags:
- azure
- sql
- sql server
---

This blog post will take you through the process of creating a backup of a SQL Azure database and restore it to a LocalDB database on your computer using the SqlPackage tool.

## Installing the SqlPackage tool

Before you get started, you will need to ensure that you have the SqlPackage tool installed. In my case, this was installed as part of Visual Studio 2019, and I could find the `sqlpackage.exe` tool under the `C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\IDE\Extensions\Microsoft\SQLDB\DAC\150` folder. This may be different for you, but, given that you have chosen to install the SQL Server tools with Visual Studio, you'll probably find in a similar folder.

If you cannot find it, you can [follow this blog post](https://docs.microsoft.com/en-us/sql/tools/sqlpackage-download?view=sql-server-2017) to download and install it.

## Creating the backup on Azure

To export the database, you can go the **Overview** page for your database in the Azure Portal. Click on the **Export** link at the top:

![](/images/blog/2019-07-06-restore-sql-database-localdb/sql-azure-database-overview.png)

This will open a blade that will allow you to configure the export. Supply the settings as required, and click on **OK**.

![](/images/blog/2019-07-06-restore-sql-database-localdb/export-database-settings.png)

This will schedule a job that will export the database and store it in the Blob container as per your export settings. You can track the progress of the export by going to the **Import/Export History** screen of your database server.

![](/images/blog/2019-07-06-restore-sql-database-localdb/export-history.png)

Once the export has completed you can download it from the container you specified in the export settings. In my case, I used the Azure Storage Explorer to do the download.

![](/images/blog/2019-07-06-restore-sql-database-localdb/download-backup.png)

## Restore the export

Now that you have downloaded the export, you can use the SqlPackage command to restore it. Open your command-line of choice (Powershell in my case) and run the following command:

```text
.\sqlpackage.exe /Action:Import /SourceFile:"C:\Users\jerri\Downloads\database-backup.bacpac" /TargetConnectionString:"Data Source=(localdb)\mssqllocaldb;Initial Catalog=CloudpressBackup;Integrated Security=true;"
```

The `SourceFile` parameter should be the path to where you saved the database export on your computer. The `TargetConnectionString` should be the correct connection string for your database.

I hope this helps!