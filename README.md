![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Shrink All Data Files Across All Databases With SQL
**Post Date: May 2, 2014**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>Here's some logic that will shrink all log files across all databases. First it checks to see if the database is in Full recovery. This way we know if the shrink file operation is even warranted. If Full Recovery is detected it will shink the log files. Additionally the script will only do this against databases that are currently 'online', so it will skip any databases that are being restored, that are offline. I usually just put this in a Job and run it every hour to aggressively manage file growth on the logs.
Hope you find this helpful.</p>      


## SQL-Logic
```SQL
use master;
set nocount on
declare @checkpoint_shrinkfile varchar(max)
set @checkpoint_shrinkfile = ''
select @checkpoint_shrinkfile = @checkpoint_shrinkfile +
'use [' + [master].sys.databases.name + ']; checkpoint; waitfor delay ''00:00:01'';' + char(10) + '
declare @recovery_type_' + cast([master].sys.databases.database_id as varchar) + ' varchar(50);' + char(10) + '
set @recovery_type_' + cast([master].sys.databases.database_id as varchar) + ' = ( select recovery_model_desc from master.sys.databases where database_id = ''' + cast([master].sys.databases.database_id as varchar) + ''' );' + char(10) + '
if @recovery_type_' + cast([master].sys.databases.database_id as varchar) + ' = ''FULL'' collate Latin1_General_CI_AS_KS_WS' + char(10) + ' begin ' + char(10) + '
dbcc shrinkfile (' + cast(sys.master_files.file_id as varchar) + ', 8)' + char(10) + 'print ''The shrinkfile operation has completed for Database [' + replace(db_name([master].sys.databases.database_id), '''', '') + ']''' + char(10) + '
end
else
print ''The shrinkfile operation is not required because the Recovery Model is SIMPLE on Database: [' + replace(db_name([master].sys.databases.database_id), '''', '') + ']'';' + char(10) + '' + char(10) + '' from
[master].sys.master_files join [master].sys.databases on
[master].sys.master_files.database_id = [master].sys.databases.database_id where
[master].sys.master_files.type_desc = 'log'
and [master].sys.databases.name not in ('master', 'model', 'msdb', 'tempdb') and [master].sys.databases.state_desc = 'online';
exec (@checkpoint_shrinkfile)
```


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

  
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

