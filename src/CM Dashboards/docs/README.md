# Readme for CM SRS Dashboards

## Welcome to our awesome MEMCM Dashboards :)

This repository is a solution of dashboards and reports, for Microsoft Endpoint Configuration Manager.

A software update dashboard with five subreports is currently available. Installation can be done manually or via the included PowerShell installer.

The `SU DAS Overall Compliance` dashboard is navigable, independent of `Software Update Groups` and comes with an array of filtering options.

All subreports can be run standalone.

## Latest release

See [releases](https://SCCM.Zone/CM-SRS-Dashboards-RELEASES).

## Changelog

See [changelog](https://SCCM.Zone/CM-SRS-Dashboards-CHANGELOG).

## Credit

* Adam Weigert [`ufn_CM_GetNextMaintenanceWindow`](https://social.technet.microsoft.com/wiki/contents/articles/7870.sccm-2007-create-report-of-upcoming-maintenance-windows-by-client.aspx)

## Dashboards and Reports

* SU DAS Overall Compliance (Main Dashboard Report)
* AL Alerts
* SU Compliance by Collection
* SU Compliance by Collection - Deprecated
* SU Compliance by Device
* SU Scan Status
* SU SUP Sync Status

## Navigation Tree

```bash
.
+-- (D) SU DAS Overall Compliance
    +-- (C) Update Compliance
    |   +-- (R) SU Compliance by Collection
    |       +-- (R) SU Compliance by Device
    |
    +-- (C) Missing updates by Category
    |   +-- (R) SU Compliance by Collection
    |       +-- (R) SU Compliance by Device
    |
    +-- (C) Update Agent Scan States
    |   +-- (R) SU Scan Status
    |
    +-- (C) Overall Update Group Compliance
    |
    +-- (C) Devices Missing a Specific Update
    |   +-- (R) SU Compliance by Collection
    |       +-- (R) SU Compliance by Device
    |
    +-- (T) Critical Alerts
    |   +-- (R) AL Alerts
    |
    +-- (T) Last Successful Synchronization Time
        +-- (R) SU SUP Sync Status

## Legend
'()'  - 'to' or 'from' navigation element
'(D)' - Dashboard
'(R)' - Report
'(C)' - Chart
'(T)' - Text
```

## Prerequisites

### Software

* Microsoft Endpoint Management Configuration Manager (MEMCM) with Windows Update Services (WSUS) integration.
* Microsoft SQL Server Reporting Services (SSRS) 2017 or above.
* Microsoft SQL [Compatibility Level](https://docs.microsoft.com/en-us/sql/t-sql/statements/alter-database-transact-sql-compatibility-level?view=sql-server-ver15) 130 or above.

### SQL User Defined Funtions (UDF)

* `ufn_CM_GetNextMaintenanceWindow` helper function (Optional)

### SQL SELECT Rights for smsschm_users (CM Reporting)

* `ufn_CM_GetNextMaintenanceWindow`
* `fnListAlerts`
* `vSMS_ServiceWindow`
* `vSMS_SUPSyncStatus`

>**Notes**
> You can find the code that automatically grants SELECT rights to the functions and tables above in the `perm_CMDatabase.sql`  file.

## Installation - Automatic

Use the provided PowerShell installer. You can find the standalone repository for the installer [here](https://SCCM.Zone/Install-SRSReport-RELEASES).

```PowerShell
## Get syntax help
Get-Help .\Install-SRSReport.ps1

## Typical installation example
#  With extensions
.\Install-SRSReport.ps1 -ReportServerUri 'http://CM-SQL-RS-01A/ReportServer' -ReportFolder '/ConfigMgr_XXX/SRSDashboards' -ServerInstance 'CM-SQL-RS-01A' -Database 'CM_XXX' -Overwrite -Verbose
#  Without extensions (Permissions will still be granted on prerequisite views and tables)
.\Install-SRSReport.ps1 -ReportServerUri 'http://CM-SQL-RS-01A/ReportServer' -ReportFolder '/ConfigMgr_XXX/SRSDashboards' -ServerInstance 'CM-SQL-RS-01A' -Database 'CM_XXX' -ExcludeExtensions -Verbose
#  Extensions only
.\Install-SRSReport.ps1 -ServerInstance 'CM-SQL-RS-01A' -Database 'CM_XXX' -ExtensionsOnly -Overwrite -Verbose
```

>**Notes**
> If you don't use `Windows Authentication` (you should!) in your SQL server you can use the `-UseSQLAuthentication` switch.
> PowerShell script needs to be run as administrator.
> If you have problems installing the SQL extensions run the script on the SQL server directly and specify the `-ExtensionsOnly` switch. If this still doesn't work check out the [`Manual Installation Steps`](#Create-the-SQL-Helper-Function).

## Installation - Manual

Upload reports to SSRS, update the datasource, grant the necessary permissions and optionally install the helper function.

### Upload Reports to SSRS

* Start Internet Explorer and navigate to [`http://<YOUR_REPORT_SERVER_FQDN>/Reports`](http://en.wikipedia.org/wiki/Fully_qualified_domain_name)
* Choose a path and upload the three report files.

>**Notes**
> Reports must be placed in the same folder on the report server.

### Configure Imported Report

* Replace the [`DataSource`](https://joshheffner.com/how-to-import-additional-software-update-reports-in-sccm/) in the reports.

### Create the SQL Helper Function

The `ufn_CM_GetNextMaintenanceWindow` is needed in order to display the next maintenance window.

* Copy paste the `ufn_CM_GetNextMaintenanceWindow` in [`SSMS`](https://docs.microsoft.com/en-us/sql/ssms/sql-server-management-studio-ssms?view=sql-server-2017)
* Uncomment the `SMS region` and change the `<SITE_CODE>` in the `USE` statement to match your Site Code.
* Click `Execute` to add the `ufn_CM_GetNextMaintenanceWindow` function to your database.
* Copy paste the `perm_CMDatabase.sql` in [`SSMS`](https://docs.microsoft.com/en-us/sql/ssms/
* Click `Execute` to add the necessary permissions to your database.

> **Notes**
> You need to have access to add the function and grant SELECT on `ufn_CM_GetNextMaintenanceWindow`, `fnListAlerts`, `vSMS_ServiceWindow` and `vSMS_SUPSyncStatus` for the `smsschm_users` (MEMCM reporting).
> If the `ufn_CM_GetNextMaintenanceWindow` is not present you will get a 'Missing helper function!' instead of the next maintenance window.
> To resolve the error codes, or get more info, just hover over the table cell.