
Quartz cluster mode
===================

Objectives of this guide
---------------------

1. Quartz JDBCStore Cluster Mode Overview
2. Create the technical tables in your database
3. Configure the application.properties file
4. Check the functioning in mono and bi-cluster
5. Examples for SQL tables and application properties

Quartz JDBCStore Cluster Mode Overview
--------------------------------------
The Quartz framework has been developed for use in cluster mode, in order to run simultaneously several instances of Quartz, so that they can share jobs and also replace each other in case of failure.

This type of operation is called cluster mode, so that the different instances can synchronize with each other, it is necessary to create between them a communication channel. The Quartz Framework offers by default several synchronization methods, the one chosen for this guide is the synchronization mode using a JDBC-StoreTX, which corresponds to synchronization via JDBC using data tables stored in the database.

For more information and to see the other sync modes available refer to the documentation.
http://www.quartz-scheduler.org/documentation/quartz-2.x/configuration/

Create the technical tables in your database
--------------------------------------------
You must use the file "QuartzTechnicalTables.txt", connect to your database and execute the CREATE TABLE commands present in the file.

Configure the application.properties file
-----------------------------------------
The application.properties file must contain a number of necessary Quartz parameters to use the tables you just created to synchronize.

You will find in the resources directory of this git a sample application.properties file sample.

These parameters are as follows:
- ** scheduler.skipUpdateCheck **: If you want to skip the execution of a fast web request to determine if an updated version of Quartz is available for download. If the check is started and an update is found, it will be reported as available in the Quartz logs. You can also disable checking for updates with the system property "org.terracotta.quartz.skipUpdateCheck = true" (which you can set in your system environment or as -D on the java command line). It is recommended that you disable the check for updates for production deployments.

- ** scheduler.instanceName **: Can be any string, and the value has no meaning for the planner itself but rather serves as a mechanism for client code to distinguish planners when multiple instances are used in the same program. If you use the clustering features, you must use the same name for each instance of the cluster that is "logically" the same scheduler.

- ** scheduler.instanceId **: Can be any string, but must be unique for all planners working as if they were the same "logical" scheduler in a cluster. You can use the value "AUTO" as instanceId if you want the Id to be generated for you. Or the value "SYS_PROP" if you want the value to come from the system property "org.quartz.scheduler.instanceId".

- ** scheduler.jobFactory.class **: The name of the JobFactory class to use. The default value is 'org.quartz.simpl.SimpleJobFactory', you can try 'org.quartz.simpl.PropertySettingJobFactory'. A JobFatcory is responsible for producing instances of JobClasses. SimpleJobFactory simply calls newInstance () on the class. PropertySettingJobFactory does this too, but also sets the properties of the job bean using the contents of the SchedulerContext (since 2.0.2) and the JobDataMaps Job and Trigger.

- ** jobStore.class **: The name of the JobFactory class to use. The default value is 'org.quartz.simpl.SimpleJobFactory', you can try 'org.quartz.simpl.PropertySettingJobFactory'. A JobFatcory is responsible for producing instances of JobClasses. SimpleJobFactory simply calls newInstance () on the class. PropertySettingJobFactory does this too, but also sets the properties of the job bean using the contents of the SchedulerContext (since 2.0.2) and the JobDataMaps Job and Trigger.

- ** jobStore.driverDelegateClass **: Specify which database driver to use for clustering. The impl.jdbcjobstore.StdJDBCDelegate driver which is compatible with most databases.

- ** jobStore.dataSource **: The value of this property must be the name of a DataSource defined in the configuration properties file.

- ** jobStore.tablePrefix **: The "table prefix" property of JDBCJobStore is a string equal to the prefix given to the Quartz tables created in your database. You can have multiple sets of Quartz tables in the same database if they use different table prefixes.

- ** jobStore.isClustered **: Set to "true" to enable cluster functionality. This property must be set to true if multiple instances of Quartz use the same set of database tables.

- ** threadPool.class **: Is the name of the ThreadPool implementation that you want to use. The thread pool provided with Quartz is "org.quartz.simpl.SimpleThreadPool" and should meet the needs of almost all users. It has a very simple behavior and is very well tested. It provides a fixed-size thread pool that "lives" the life of the scheduler.

- ** threadPool.threadCount **: Can be any positive integer. This is the number of threads available for running jobs concurrently. If you only have a few jobs passing a few times a day, then 1 thread is enough. If you have tens of thousands of jobs, in parallel with each minute, then a number of threads of more than 50 or 100 will be more suitable (it depends very much on the frequency, the jobs and what they realize and of course available system resources).

- ** jobStore.clusterCheckinInterval **: Set the frequency (in milliseconds) that this instance "registers" * with the other instances of the cluster. Affects the speed of detection of failed instances.

- ** quartzDataSource.URL **: The connection URL (host, port, etc.) for connecting to your database.

- ** quartzDataSource.user **: The username to use when connecting

Check the functioning in mono and bi-cluster
--------------------------------------------

When we test our java developments, we only have one server. This is not a problem for using Quartz in cluster mode because it works very well in this mode with a single server.

You just have to start your freecaster and use DBeaver to find that your freecaster is writing the information in the tables you created.

To verify cluster mode operation, you must another server and pay particular attention to the fact that both server must use the same database.

Examples for SQL tables and application properties
--------------------------------------------------

The resources directory contains an example of the parameters to place in the properties of your application

The createQuartzTables directory contains a sample script for creating the tables needed to run Quartz
