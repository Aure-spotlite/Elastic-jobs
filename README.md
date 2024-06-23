# Elastic-jobs
Elastic jobs creation step to step process
we have elastic jobs in DEV & PROD SERVERS

PRODUCTIOIN SERVER ELASTIC JOBS DETAILS
@CredentialName  = N'JobRun';
@TargetGroupName = N'Production';

####################################################################################
select * from jobs.jobsteps where job_name='Dashboard Environment Daily_indiayodev'
select * from jobs.jobsteps where job_name='Dashboard Environment Daily'

select * from jobs.jobsteps where target_group_name='development'

select * from jobs.jobsteps where target_group_name='indiayodev'

--displays total number jobs in database
select * from jobs.jobs

--displays total groups in jobs database
select * from jobs.target_groups

--displays specified target group members
select * from jobs.jobsteps where target_group_name='indiayodev'

--displays stepname, command, credentails and target group name
select * from jobs.jobsteps where job_name='Water History Daily Data Transfer'

--displays discription , interval, schedule date,end date, Enabled value
select * from jobs.jobs where job_name='Water History Daily Data Transfer'

----View all execution statuses for job 
SELECT * FROM jobs.job_executions
WHERE job_name = 'End Point Data History Daily Data Transfer' ORDER BY start_time DESC

--View the recently added database in target group
SELECT
      target_group_name
    , membership_type
    , target_type
    , server_name
    , [database_name]
FROM jobs.target_group_members 
WHERE target_group_name='devindianimbledb';

--SQL Script used to find the database name, server name, target group id
SELECT * FROM jobs.target_groups
SELECT target_group_name, 
        membership_type,
        refresh_credential_name,
        server_name,
        database_name
FROM jobs.target_group_members

--disable the jobs
exec jobs.sp_update_job
@job_name='Endpoint History:PROD',
@enabled=0
--disable =0 used to stop the jobs
--enable =1 used to run the jobs


--Delete the jobs
EXEC jobs.sp_delete_job @job_name='Alarms Transfer_dev_in';

--Delete the target group
EXEC [jobs].sp_delete_target_group @target_group_name = 'MyTargetGroup';

--Used to stop the running job in progess
EXEC jobs.sp_stop_job '0BBDE512-263B-4EC0-B6B3-83C5A169FA96';

--Used to find the scoped credentials on jobs DB
SELECT * FROM sys.database_scoped_credentials

--Used to update/ Alter the scheduled interval time for 15 minutes
--ctrl+f = update search as you can find the details for update the job
ref: https://docs.microsoft.com/en-us/azure/azure-sql/database/elastic-jobs-tsql-create-manage?view=azuresql
EXEC jobs.sp_update_job
@job_name = 'WorkflowSLABreach_devnimble',
@enabled=1,
@schedule_interval_type = 'Minutes',
@schedule_interval_count = 15;

--Used to update the job details
exec [jobs].sp_update_job [ @job_name = ] 'job_name'  
  [ , [ @new_name = ] 'new_name' ]
  [ , [ @description = ] 'description' ]
  [ , [ @enabled = ] enabled ]
  [ , [ @schedule_interval_type = ] schedule_interval_type ]  
  [ , [ @schedule_interval_count = ] schedule_interval_count ]
  [ , [ @schedule_start_time = ] schedule_start_time ]
  [ , [ @schedule_end_time = ] schedule_end_time ]
  
  
  --Used to update the Time interval of the job.
  --Script used to change the schedule start time from 2023-02-24 to 2023-01-01
EXEC jobs.sp_update_job
@job_name='Water History Monthly Data Transfer_dev',
@enabled=1,
@schedule_interval_type='Months',
@schedule_interval_count=1,
@schedule_start_time='2023-01-01 00:00:00'
  
--Used to delete the jobs force fully, whenjob doesn't not stop through disable=0, still need to drop immediately use below query. 
 exec jobs.sp_delete_job   'WorkflowSLABreach_dev'
   , @force = 1


--Used to get data from both job_steps&jobs schedule time cred&db name
SELECT  js.job_name,j.schedule_interval_type,j.schedule_interval_count,js.step_name,js.command,js.credential_name,js.target_group_name
 FROM jobs.jobsteps js
JOIN jobs.jobs j
  ON j.job_id = js.job_id AND j.job_version = js.job_version;
  
STORED PROCEDURE		DESCRIPTION
sp_add_job				Adds a new job.
sp_update_job			Updates an existing job.
sp_delete_job			Deletes an existing job.
sp_add_jobstep			Adds a step to a job.
sp_update_jobstep		Updates a job step.
sp_delete_jobstep		Deletes a job step.
sp_start_job			Starts executing a job.
sp_stop_job				Stops a job execution.
sp_add_target_group		Adds a target group.
sp_delete_target_group	Deletes a target group.
sp_add_target_group_member	Adds a database or group of databases to a target group.
sp_delete_target_group_member	Removes a target group member from a target group.
sp_purge_jobhistory		Removes the history records for a job.


--Used to find the Master key available or not in the dtaabase.
SELECT * FROM SYS.symmetric_keys

--USED TO GET USERNAME: JobUser & CREDENTIAL: JobRun & CREATED DATE
SELECT * FROM SYS.database_credentials


-- TO DELETE HISTORY FOR A SPECIFIC JOB UP TO SPECIFIC DATE
--It will delete only the history of the jobs not data in the tables or in main database
syntax: 
EXEC dbo.sp_purge_jobhistory
@job_name = N'<job name>' ,
@oldest_date = '<date>'


--TO SCHEDULE MAINTENANCE JOBS IN AZURE SQL SERVER ON SUNDAY AT 00:00:00 USING AZURE LOGIC APP
Ref: https://learn.microsoft.com/en-us/answers/questions/748092/elastic-job-schedule
Above ref is same requirement but no exact data
How to schedule elastic jobs only on sundays like maintenance jobs.
--Use recurrence trigger in logic app to run the stored procedure on saturday london time zone at 00:00:00
we can select all this options in recurrence trigger .. 
2.Add exec stored procedure in sql server is a connector in Azure logic app to execute

Syntax: SCHEDULING AT 2:00 AM
EXEC jobs.sp_update_job
  @job_name = 'IndexOptimizeDB'
  ,@enabled = 1
  ,@schedule_interval_type = 'Days'
  ,@schedule_start_time = '20200301 02:00:00' --HERE TIME IS INCLUDING WE ARE EXEC AT 2:00 AM
  ,@schedule_end_time = '9999–12–31 11:59:59.0000000';
GO


SIMPLY DEFINE A DEFAULT VALUE FOR THAT PARAMETER. 
ELASTIC JOBS ARE NOT ACCEPTING VARIABLE IN COMMAND LINE WITH SINGLE QUOTE WITHOUT PARAM NAME
USE BELOW CASE TO SOLVE THE ISSUE.
Ref: https://medium.com/@yuvrendergill21/sql-stored-procedures-parameters-4e315365adfd#:~:text=To%20execute%20procedure%20with%20parameters,clause%20followed%20by%20procedure%20name.&text=To%20add%20more%20than%20one,parameters%20followed%20by%20the%20first.
Syntax:
EXEC SP_LIST_COLUMN_1 @Parameter1=100, @Parameter2=50, @Parameter3='Toronto'

WE HAVE PASSED PARAMETERS TO STORED PROCEDURE IN ELASTIC JOBS WITH DEPAULT DECLARED NAME
SYNTAX:
exec usp_CreateTableFromView @tableName=CombinedMonthlyReportGraphGEN, @viewName=CombinedMonthlyReportAllStatsGraphView'

--Step-7 Adding command line& step name on Jobs DB
DECLARE @JobStepName NVARCHAR(128) = N'CombinedMonthlyReportFMGEN & WorkFlowAllTicketReportsFMView';
DECLARE @Command NVARCHAR(MAX) = N'EXEC usp_CreateTableFromView @tableName=CombinedMonthlyReportFMGEN,@viewName=WorkFlowAllTicketReportsFMView'-- // executing stored procedure
DECLARE @CredentialName NVARCHAR(128) = N'JobRun';
DECLARE @TargetGroupName NVARCHAR(128) = N'Production';
DECLARE @retryAttempts INT = 3; --retry attempts on failure

EXEC jobs.sp_add_jobstep @job_name = N'CombinedMonthlyReportFMGEN & WorkFlowAllTicketReportsFMView prod',

@step_name = @JobStepName,
@command = @Command,
@credential_name = @CredentialName,
@target_group_name = @TargetGroupName,
@retry_attempts = @retryAttempts;

--SCRIPT USED TO FIND THE TARGET_ID 
--This target ID id ued to delete the target group & member
--Here you can see some null values based on this we can confirm target group & member target_id
--Expand niblesql_jobs DB we have SP to delete the target group & member target_id
SELECT  target_id,target_group_name,server_name,database_name  
    FROM [jobs_internal].targets  


SCHEDULE_INTERVAL_TYPES IN ELASTIC JOBS:
Ref: https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-jobs-tsql-create-manage?view=azuresql
@schedule_interval_type =] schedule_interval_type
--copy and paste this exactly to exec correctly
'Once',
'Minutes',
'Hours',
'Days',
'Weeks',
'Months'

*******************************************************************************************
PROCESS TO CREATE ELASTIC JOBS
*******************************************************************************************
Ref: https://apiumhub.com/tech-blog-barcelona/azure-elastic-jobs-for-sql-databases/

Ref used to study the purpose & detailed explanation of ELASTIC JOBS.
Ref: https://www.sqlshack.com/elastic-jobs-in-azure-sql-database/

Use this referance explained which command to be used on whic DB.
Ref:https://apiumhub.com/tech-blog-barcelona/azure-elastic-jobs-for-sql-databases/

Elastic Jobs in Azure SQL Database can be utilized to execute the same query in multiple Azure SQL Databases parallelly.
 Elastic jobs are used to run the schedule task. Elastic jobs can run the scheduled task in multiple servers & multiple databases at a time.
 Used to monitor the stats time, updat , delete tasks and scheduled time of the data.
1st configure the  Elastic job agent & it requires atleast S0, atleast requires 10 DTU's to run the Elastic jobs.
select the jobs database while configuring the elastic job agent .
1.Use same password for both scoped credentials & login password
2.scoped credential userid,password and login_ID & password must be same to create an elastic jobs.
3.There is no need to be same masterkey name & password it should be same/diffrent  no problem but
4.scoped credential & login_ID & password must be same.

--Step 0 create master key on JOBS DATABSE.
Already master key created so no need again
create master key encryption by password = 'StrongPassword123!'
go


--Step1 To create SCOPED CREDENTIAL on Jobs database
--Make sure scoped credentials user name & password   must  matchwith login, user on  DB.
--Whenever new server is adding need to create the scoped cred on Jobs DB and create login add the job
CREATE DATABASE SCOPED CREDENTIAL jobuserindia WITH IDENTITY = 'jobuserindia',
SECRET = 'jobuserindia@123';  
GO
-- To change secret password Alter command used
--Same password used for jobuserLon1SB 
ALTER DATABASE SCOPED CREDENTIAL jobuserindia WITH IDENTITY ='jobuserindia',
SECRET='Uxkguv2KUy6SKr8XetBy'

--Step 2 To create Login on Master database(exec on Master DB)
CREATE LOGIN jobuserindia WITH PASSWORD = 'Uxkguv2KUy6SKr8XetBy';

--step3 creating user on Indian DB, dev_nimble user is created on which we want to insert data.
create user jobuserindia from login jobuserindia
EXEC sys.sp_addrolemember 'db_owner','jobuserNovaSB'


--step4 Adding Target group on jobs DB
EXEC jobs.sp_add_target_group N'indiayodev'
GO

--step5 Adding Target Group Members on Jobs DB
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'indiayodev',
@target_type = N'SqlDatabase',
@server_name = N'sqlyondr-india-dcops1.database.windows.net',
@database_name =N'db-india-yo-dev-yondr0';

--STEP 6 Creating Jobs on Jobs DB
DECLARE @JobName NVARCHAR(128) = N'End Point Data History Hourly Data Transfer_indiayodev';
DECLARE @JobDescription NVARCHAR(512) = N'Populates Hourly EndPoint Data History';
DECLARE @Enabled BIT = 1;
DECLARE @ScheduleIntercalType NVARCHAR(50) = N'Hours';
DECLARE @ScheduleIntervalCount INT = 1;
DECLARE @ScheduleStart DATETIME2 = N'20200801 00:00:00';--yyyymmdd hh:mm:ss--2020-08-24 11:52:00.0000000
EXEC jobs.sp_add_job @job_name = @JobName,
@description = @JobDescription,
@enabled = @Enabled,
@schedule_interval_type = @ScheduleIntercalType,
@schedule_interval_count = @ScheduleIntervalCount,
@schedule_start_time = @ScheduleStart;

--Step-7 Adding command line& step name on Jobs DB
DECLARE @JobStepName NVARCHAR(128) = N'End Point Data History Hourly Data population_indiayodev';
DECLARE @Command NVARCHAR(MAX) = N'exec usp_EndPointDataHistoryHour;'-- // executing stored procedure
DECLARE @CredentialName NVARCHAR(128) = N'jobuserindia';
DECLARE @TargetGroupName NVARCHAR(128) = N'indiayodev';
DECLARE @retryAttempts INT = 3; --retry attempts on failure

EXEC jobs.sp_add_jobstep @job_name = N'End Point Data History Hourly Data Transfer_indiayodev',

@step_name = @JobStepName,
@command = @Command,
@credential_name = @CredentialName,
@target_group_name = @TargetGroupName,
@retry_attempts = @retryAttempts;

--step 8 Exec jobs manually on Indian DB
EXEC jobs.sp_start_job @job_name = N'End Point Data History Hourly Data Transfer_indiayodev';

--step 9  on master database
EXEC sp_addrolemember 'db_owner', 'jobuserindia';

--step 10
--use master DB to get details
select * from sys.firewall_rules

--Execute it on Master DB, below Ip address is displayed as error login failed by firewall rule, add that ip address.
EXECUTE sp_set_firewall_rule @name = N'client_ip_address_azure_elsatic_jobs',
   @start_ip_address = '51.140.144.10', @end_ip_address = '51.140.144.10'
   
--step 11 open Indian DB ,views, jobs.execution rt.click SelectTop 1000 rows
/****** Script for SelectTopNRows command from SSMS  ******/
SELECT TOP (1000) [job_execution_id]


--due to Indian DB is deleted so jobs are failing & then this jobs are replacing to dev_nimble by adding DB then IndiaYoDev target group is deleted
--used to add target group to diffrent data base before deleting previous target 
--step5 is repeating to add DB by adding other DB same jobs are running on dev_nimble as well added new DB

EXEC [jobs].sp_add_target_group_member
@target_group_name = N'indiayodev',
@target_type = N'SqlDatabase',
@server_name = N'sqlyondr-india-dcops1.database.windows.net',
@database_name =N'dev_nimble';


--Syntax is used to delete target group member

EXEC jobs.sp_delete_target_group_member
@target_group_name='indiayodev',
@target_id='C91C1730-DECE-49DC-8BE2-22B6F3EC8946'


--Like is used to filter the data 
select * from jobs.jobs where job_name like '%water%'



------------------------------------------------------------
ISSUE: IN PRODUCTION SERVER
  Command failed: The EXECUTE permission was denied on the object 'usp_CreateTableFromView',
  database 'db-uks-yo-prod-yondrone-001', schema 'dbo'. (Msg 229, Level 14, State 5, Line 1)
  
--USED TO GET USERNAME: JobUser & CREDENTIAL: JobRun & CREATED DATE
SELECT * FROM SYS.database_credentials

CHECK THIS USERNAME: JobUser AVAILABLE IN MASTER & PROD DATABASE
IF AVAILABLE GIVE DB_OWNER PERMISSIONS ELSE ADD THE USER.

Here we have missed permissions
SYNTAX:
Exec sp_addrolemember 'db_owner','JobUser';
------------------------------------------------------------
creating jobs on production

--STEP 6 Creating Jobs on Jobs DB
DECLARE @JobName NVARCHAR(128) = N'End Point Data History Daily Data Transfer_prod_new';
DECLARE @JobDescription NVARCHAR(512) = N'Populates Daily EndPoint Data History';
DECLARE @Enabled BIT = 1;
DECLARE @ScheduleIntercalType NVARCHAR(50) = N'Days';
DECLARE @ScheduleIntervalCount INT = 1;
DECLARE @ScheduleStart DATETIME2 = N'2020-08-24 23:52:00.0000000';--yyyymmdd hh:mm:ss--2020-08-24 11:52:00.0000000
EXEC jobs.sp_add_job @job_name = @JobName,
@description = @JobDescription,
@enabled = @Enabled,
@schedule_interval_type = @ScheduleIntercalType,
@schedule_interval_count = @ScheduleIntervalCount,
@schedule_start_time = @ScheduleStart;

--Step-7 Adding command line& step name on Jobs DB
DECLARE @JobStepName NVARCHAR(128) = N'End Point Data History Daily Data population';
DECLARE @Command NVARCHAR(MAX) = N'exec usp_EndPointDataHistoryDaily'-- // executing stored procedure
DECLARE @CredentialName NVARCHAR(128) = N'JobRun';
DECLARE @TargetGroupName NVARCHAR(128) = N'Production';
DECLARE @retryAttempts INT = 3; --retry attempts on failure

EXEC jobs.sp_add_jobstep @job_name = N'End Point Data History Daily Data Transfer_prod_new',

@step_name = @JobStepName,
@command = @Command,
@credential_name = @CredentialName,
@target_group_name = @TargetGroupName,
@retry_attempts = @retryAttempts;



--disable the jobs
--I have tested it working well.

exec jobs.sp_update_job
@job_name='Endpoint History:PROD',
@enabled=0

0= Disable to stop job
1= Enable to run job


************************************************************
Databases-DEV Server
host:       nimblyone.database.windows.net, DB: dev_nimble
************************************************************
step 1:
CREATE DATABASE SCOPED CREDENTIAL devindia WITH IDENTITY = 'devindia',
SECRET = 'devindia@123';  

step 2: execute on master DB
CREATE LOGIN devindia WITH PASSWORD = '28wAYLC4v6g9KhiC';

step 3:
create user devindia from login devindia

--step4 Adding Target group on jobs DB
EXEC jobs.sp_add_target_group N'devindianimbledb'
GO

--step5 Adding Target Group Members on Jobs DB
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'devindianimbledb',
@target_type = N'SqlDatabase',
@server_name = N'nimblyone.database.windows.net',
@database_name =N'dev_nimble';

--STEP 6 Creating Jobs on Jobs DB
DECLARE @JobName NVARCHAR(128) = N'ThresholdEndpointBreach_devindianimbledb';
DECLARE @JobDescription NVARCHAR(512) = N'Populates thresholds notification exceeded endpoint values';
DECLARE @Enabled BIT = 1;
DECLARE @ScheduleIntercalType NVARCHAR(50) = N'Minutes';
DECLARE @ScheduleIntervalCount INT = 1;
DECLARE @ScheduleStart DATETIME2 = N'20200824 00:00:00';--yyyymmdd hh:mm:ss--2020-08-24 11:52:00.0000000
EXEC jobs.sp_add_job @job_name = @JobName,
@description = @JobDescription,
@enabled = @Enabled,
@schedule_interval_type = @ScheduleIntercalType,
@schedule_interval_count = @ScheduleIntervalCount,
@schedule_start_time = @ScheduleStart;



--Step-7 Adding command line& step name on Jobs DB
DECLARE @JobStepName NVARCHAR(128) = N'ThresholdEndpointBreach population_devindianimbledb';
DECLARE @Command NVARCHAR(MAX) = N'exec ThresholdEndpointBreach'-- // executing stored procedure
DECLARE @CredentialName NVARCHAR(128) = N'devindia';
DECLARE @TargetGroupName NVARCHAR(128) = N'devindianimbledb';
DECLARE @retryAttempts INT = 3; --retry attempts on failure

EXEC jobs.sp_add_jobstep @job_name = N'ThresholdEndpointBreach_devindianimbledb',

@step_name = @JobStepName,
@command = @Command,
@credential_name = @CredentialName,
@target_group_name = @TargetGroupName,
@retry_attempts = @retryAttempts;


--step 8 Exec jobs manually on Indian DB
EXEC jobs.sp_start_job @job_name = N'ThresholdEndpointBreach_devindianimbledb';

*****************************************************************************************
--(schedule_jobs_specific_time)USED TO SCHEDULE THE JOB AT SPECIFIC TIME EX:DAILY AT 00:30 IN MIDNIGHT 
--step1:
DECLARE @JobName NVARCHAR(128) = N'notifications_test_sonarqdb';
EXEC jobs.sp_add_job @job_name = @JobName
 
 --step2:
EXEC jobs.sp_update_job
@job_name='notifications_test_sonarqdb',
@enabled=1,
@schedule_interval_type='Days',
@schedule_interval_count=1,
@schedule_start_time='2023-03-03 14:14:00' --(use UTC time here to schedule in specific time getdate())


--Step-3 Adding command line& step name on Jobs DB
DECLARE @JobStepName NVARCHAR(128) = N'Testing at time wise schedule';
DECLARE @Command NVARCHAR(MAX) = N'exec test_timebasedexec'-- // executing stored procedure
DECLARE @CredentialName NVARCHAR(128) = N'JobRun';
DECLARE @TargetGroupName NVARCHAR(128) = N'Sonarq';
DECLARE @retryAttempts INT = 3; --retry attempts on failure

EXEC jobs.sp_add_jobstep @job_name = N'notifications_test_sonarqdb',

@step_name = @JobStepName,
@command = @Command,
@credential_name = @CredentialName,
@target_group_name = @TargetGroupName,
@retry_attempts = @retryAttempts;

*****************************************************************************************

--Step 2 To create Login on Master database
CREATE LOGIN jobuser_devdb WITH PASSWORD = '6B485aceiYtvX4b';
Alter LOGIN jobuser_devdb WITH PASSWORD = '7hco84Mf2Ia2ZOt';

--step3 creating user on Indian DB,dev_nimble
--Execute it on database which we want to insert the data after SP execute 
create user jobuser_devdb from login jobuser_devdb


--Step1 To create SCOPED CREDENTIAL on Jobs database
--create on jobs database
CREATE DATABASE SCOPED CREDENTIAL jobuser_devdb WITH IDENTITY = 'jobuser_devdb',
SECRET = 'Jobuser_devdb@123';  
GO

ALTER DATABASE SCOPED CREDENTIAL jobuser_devdb WITH IDENTITY = 'jobuser_devdb',
SECRET = '7hco84Mf2Ia2ZOt'; 

--step4 Adding Target group on jobs DB
EXEC jobs.sp_add_target_group N'dev_nimble_jobs'
GO



--step5 Adding Target Group Members on Jobs DB
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'dev_nimble_jobs',
@target_type = N'SqlDatabase',
@server_name = N'nimblyone.database.windows.net',
@database_name =N'dev_nimble';


--STEP 6 Creating Jobs on Jobs DB
DECLARE @JobName NVARCHAR(128) = N'WorkflowSLABreach_devnimble';
DECLARE @JobDescription NVARCHAR(512) = N'Populates Notification';
DECLARE @Enabled BIT = 1;
DECLARE @ScheduleIntercalType NVARCHAR(50) = N'Minutes';
DECLARE @ScheduleIntervalCount INT = 1;
DECLARE @ScheduleStart DATETIME2 = N'20200824 00:00:00';--yyyymmdd hh:mm:ss--2020-08-24 11:52:00.0000000
EXEC jobs.sp_add_job @job_name = @JobName,
@description = @JobDescription,
@enabled = @Enabled,
@schedule_interval_type = @ScheduleIntercalType,
@schedule_interval_count = @ScheduleIntervalCount,
@schedule_start_time = @ScheduleStart;


DECLARE @JobStepName NVARCHAR(128) = N'WorkflowSLABreach population_devdb';
DECLARE @Command NVARCHAR(MAX) = N'exec WorkflowSLABreach'-- // executing stored procedure
DECLARE @CredentialName NVARCHAR(128) = N'jobuser_devdb';
DECLARE @TargetGroupName NVARCHAR(128) = N'dev_nimble_jobs';
DECLARE @retryAttempts INT = 3; --retry attempts on failure

EXEC jobs.sp_add_jobstep @job_name = N'WorkflowSLABreach_devnimble',

@step_name = @JobStepName,
@command = @Command,
@credential_name = @CredentialName,
@target_group_name = @TargetGroupName,
@retry_attempts = @retryAttempts;


--step 8 Exec jobs manually on Indian DB
EXEC jobs.sp_start_job @job_name = N'WorkflowSLABreach_devnimble';

select * from jobs.job_executions where job_name = 'WorkflowSLABreach_devnimble'


--step 9 
EXEC sp_addrolemember 'db_owner', 'jobuser_devdb';

workflow sla breach_devindianimbledb	--target_group_name='devindianimbledb'
@CredentialName NVARCHAR(128) = N'devindia';
@TargetGroupName NVARCHAR(128) = N'devindianimbledb';

************************************************************
Indian server: sqlyondr-india-dcops1.database.windows.net
*************************************************************
--select * from jobs.jobsteps where target_group_name= 'indiayodev'
-- db-india-yo-dev-yondr0--deleted, Server: sqlyondr-india-dcops1.database.windows.net
BMS data transfer_indiayodev -- 
Dashboard Environment Daily_indiayodev

--nimblyone.database.windows.net, sonarq
Dashboard Environment Daily_Sonarq



--step5 Adding Target Group Members on Jobs DB
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'indiayodev',
@target_type = N'SqlDatabase',
@server_name = N'sqlyondr-india-dcops1.database.windows.net',
@database_name =N'dev_nimble';

--STEP 6 Creating Jobs on Jobs DB
DECLARE @JobName NVARCHAR(128) = N'Threshold Management Sync Active Alarm To Notifications_indiayodev';
DECLARE @JobDescription NVARCHAR(512) = N'Populates Notifications';
DECLARE @Enabled BIT = 1;
DECLARE @ScheduleIntercalType NVARCHAR(50) = N'Minutes';
DECLARE @ScheduleIntervalCount INT = 1;
DECLARE @ScheduleStart DATETIME2 = N'20200824 00:00:00';--yyyymmdd hh:mm:ss--2020-08-24 11:52:00.0000000
EXEC jobs.sp_add_job @job_name = @JobName,
@description = @JobDescription,
@enabled = @Enabled,
@schedule_interval_type = @ScheduleIntercalType,
@schedule_interval_count = @ScheduleIntervalCount,
@schedule_start_time = @ScheduleStart;

--Step-7 Adding command line& step name on Jobs DB
DECLARE @JobStepName NVARCHAR(128) = N'Threshold Management Sync Active Alarm To Notifications population_indiayodev';
DECLARE @Command NVARCHAR(MAX) = N'exec ThresholdManagementSyncActiveAlarmToNotifications'-- // executing stored procedure
DECLARE @CredentialName NVARCHAR(128) = N'jobuserindia';
DECLARE @TargetGroupName NVARCHAR(128) = N'indiayodev';
DECLARE @retryAttempts INT = 3; --retry attempts on failure

EXEC jobs.sp_add_jobstep @job_name = N'Threshold Management Sync Active Alarm To Notifications_indiayodev',

@step_name = @JobStepName,
@command = @Command,
@credential_name = @CredentialName,
@target_group_name = @TargetGroupName,
@retry_attempts = @retryAttempts;

--step 8 Exec jobs manually on Indian DB
EXEC jobs.sp_start_job @job_name = N'Threshold Management Sync Active Alarm To Notifications_indiayodev';


select  job_name,lifecycle,create_time,last_message from jobs.job_executions where 
job_name='Threshold Management Sync Active Alarm To Notifications_indiayodev' order by  create_time desc


*****************************************************************
Test_job Database jobs
**************************************************************

CREATE LOGIN TestJobuser WITH PASSWORD='qazwsx!143';

--Use Master
CREATE USER TestJobuser FROM LOGIN TestJobuser;

--Use Test_Job
CREATE USER TestJobuser FROM LOGIN TestJobuser;


EXEC sp_addrolemember 'db_owner', 'TestJobuser';

--To check existing master key --SELECT * FROM sys.symmetric_keys


--Creation of Master Key
CREATE MASTER KEY ENCRYPTION BY PASSWORD='qazwsx!143';  

-- To Drop Master key --DROP MASTER KEY 

--To check DB scoped credentials:
SELECT * FROM sys.database_scoped_credentials

--Create Db Scoped Credentails:
CREATE DATABASE SCOPED CREDENTIAL TestJobuser

WITH IDENTITY = 'TestJobuser',

SECRET = 'qazwsx!143';

--Creation of Target Group
EXEC jobs.sp_add_target_group 'TestGroup';

--Creating Target group and members for Azure SQL DB:
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'TestGroup',
@target_type = N'SqlDatabase',
--@refresh_credential_name='TestJobuser',
@server_name = N'nimblyone.database.windows.net',
@database_name =N'Test_job';

--Creating Job and Job steps:
EXEC jobs.sp_add_job @job_name='CreateTableTest', @description='Create Table Test'

EXEC jobs.sp_add_jobstep @job_name='CreateTableTest',
@command=N'IF NOT EXISTS (SELECT * FROM sys.tables WHERE object_id = object_id(''Test''))
CREATE TABLE [dbo].[Test]([TestId] [int] NOT NULL);',
@credential_name='TestJobuser',
@target_group_name='TestGroup'

--Command Line
EXEC jobs.sp_add_jobstep @job_name='CreateTableTest',
@command=N'IF NOT EXISTS (SELECT * FROM sys.tables WHERE object_id = object_id(''Test''))
CREATE TABLE [dbo].[Test]([TestId] [int] NOT NULL);',
@credential_name='TestJobuser',
@target_group_name='TestGroup'


**********************************************************************************************
ISSUE: DB_renamed so jobs failed, deleted old target group member and add new (DB name)
**********************************************************************************************
1.We had to rename a database, which caused the elastic jobs to fail.(dev_nimble to yondrone_core_db)

2.OPen the Elastic jobs database and identified the failed jobs.
--used to find the failed elastic jobs in the jobsDB
select  distinct job_name
 from jobs.job_executions where lifecycle='failed' and create_time>=dateadd(day,-1,getdate())
 
3.Then located the target group name, target group member, and credential name for each failed job.
--Used to find the target group name, traget group member name
SELECT target_group_name, 
        membership_type,
        refresh_credential_name,
        server_name,
        database_name
FROM jobs.target_group_members


--script used to find the target_id 
--This target ID id ued to delete the target group & member
--Here you can see some null values based on this we can confirm target group & member target_id
SELECT  target_id,target_group_name,server_name,database_name  
    FROM [jobs_internal].targets  
	
	
4.To resolve the issue, we executed the failed jobs and copied the ID from the "target_id" column.
--Used to find the target_id NOTE: target_ID is diff from target group id
select job_name,lifecycle,create_time,last_message,target_id
 from jobs.job_executions 
where job_name='Weather Data Transfer_indiayodev' and create_time>=dateadd(day,-1,getdate())
order by create_time desc

5.Next, we deleted the target group member from the target group.
--Used to delete the target group member(be carefull before delete check ID is related or not)
EXEC jobs.sp_delete_target_group_member
@target_group_name='indiayodev',
@target_id='5C542D54-924D-4EB8-AC7A-96D9A0797DFF'

6.After deleting, we added the target group member back with the updated database name.
--used to  Adding Target Group Members on Jobs DB
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'ownerchangedb',
@target_type = N'SqlDatabase',
@server_name = N'devserverpractice.database.windows.net',
@database_name =N'ownerchange';


ISSUE: procuction server elastic jobs
PRODUCTION JOBS ARE CREATED, SCHEDULED AT SPECIFIC TIME  1st OF EVERY MONTH  02:00:00  IN UK TIMING
--TIME DIFFRENCE IS 4:30 MINUTES (open harry teams it will display time of Harry location)
--(schedule_jobs_specific_time)Used to schedule the job at specific time ex:daily at 00:30 in midnight 
--step1:
DECLARE @JobName NVARCHAR(128) = N'Report Ranges merge';
EXEC jobs.sp_add_job @job_name = @JobName
 
 --step2:
EXEC jobs.sp_update_job
@job_name='Report Ranges merge',
@enabled=1,
@schedule_interval_type='Months',
@schedule_interval_count=1,
@schedule_start_time='2023-05-01 01:00:00'  --(use UTC time here to schedule in specific time getdate())

--select getdate() --2023-04-03 02:00:00

--Step-3 Adding command line& step name on Jobs DB
DECLARE @JobStepName NVARCHAR(128) = N'Report Ranges table data';
DECLARE @Command NVARCHAR(MAX) = N'exec USP_ReportRanges_merge'-- // executing stored procedure
DECLARE @CredentialName NVARCHAR(128) = N'JobRun';
DECLARE @TargetGroupName NVARCHAR(128) = N'Production';
DECLARE @retryAttempts INT = 3; --retry attempts on failure

EXEC jobs.sp_add_jobstep @job_name = N'Report Ranges merge',

@step_name = @JobStepName,
@command = @Command,
@credential_name = @CredentialName,
@target_group_name = @TargetGroupName,
@retry_attempts = @retryAttempts;




---Prod server --

CREATE DATABASE SCOPED CREDENTIAL devDB_jobs WITH IDENTITY = 'devDB_jobs',
SECRET = '6P80yMbtfQZcEH64'; 


 CREATE DATABASE SCOPED CREDENTIAL dev_in WITH IDENTITY = 'dev_in',
    SECRET ='567KKW7MjJhsr2Yd';#
	
--step4 Adding Target group on jobs DB
EXEC jobs.sp_add_target_group N'dev_in'
GO

--step5 Adding Target Group Members on Jobs DB
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'dev_in',
@target_type = N'SqlDatabase',
@server_name = N'sqlyondr-india-dcops1.database.windows.net',
@database_name =N'dev_nimble';

--STEP 6 Creating Jobs on Jobs DB
DECLARE @JobName NVARCHAR(128) = N'Alarms Transfer_dev_in';
DECLARE @JobDescription NVARCHAR(512) = N'Populates Alarms table from activealaram view& AlarmSuppression';
DECLARE @Enabled BIT = 1;
DECLARE @ScheduleIntercalType NVARCHAR(50) = N'Minutes';
DECLARE @ScheduleIntervalCount INT = 1;
DECLARE @ScheduleStart DATETIME2 = N'20200824 00:00:00';--yyyymmdd hh:mm:ss--2020-08-24 11:52:00.0000000
EXEC jobs.sp_add_job @job_name = @JobName,
@description = @JobDescription,
@enabled = @Enabled,
@schedule_interval_type = @ScheduleIntercalType,
@schedule_interval_count = @ScheduleIntervalCount,
@schedule_start_time = @ScheduleStart;

--Step-7 Adding command line& step name on Jobs DB
DECLARE @JobStepName NVARCHAR(128) = N'Alarms table from activealaram view& AlarmSuppression population dev_in';
DECLARE @Command NVARCHAR(MAX) = N'exec usp_AlarmsTransfer;'-- // executing stored procedure
DECLARE @CredentialName NVARCHAR(128) = N'dev_in';
DECLARE @TargetGroupName NVARCHAR(128) = N'dev_in';
DECLARE @retryAttempts INT = 3; --retry attempts on failure

EXEC jobs.sp_add_jobstep @job_name = N'Alarms Transfer_dev_in',

@step_name = @JobStepName,
@command = @Command,
@credential_name = @CredentialName,
@target_group_name = @TargetGroupName,
@retry_attempts = @retryAttempts;

--step 8 Exec jobs manually on Indian DB
EXEC jobs.sp_start_job @job_name = N'End Point Data History Hourly Data Transfer_indiayodev';


CREATE DATABASE SCOPED CREDENTIAL rbk_lon1 WITH IDENTITY = 'rbk_lon1',
SECRET = 'w675q7XUY5B$';  
