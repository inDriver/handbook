# Database Migration agreements

Migrations are database update scripts whose purpose is to change the logical structure or data schema (DDL), or to update, delete or migrate any data (DML).

For data structure and schema (DDL) migrations, Liquibase should be used because this approach preserves the complete history of data schema changes. The rules for writing Liquibase migration scripts are contained in the following instructions:

- [Database migrations](https://indriver.atlassian.net/wiki/spaces/D/pages/1511227800)

- [MySQL management](https://github.com/inDriver/base-chart/wiki#mysql-management)

This convention further discusses the data migration process (DML).

## Definitions and abbreviations

**Blast radius** - list of services, tools that may be affected by the release.

**DDL** - SQL statements used to create, modify, and delete database objects, including tables, views, indexes, and stored procedures.

**DML** - SQL statements that are used to manipulate data in a database. DML Statements are used to insert, update, and delete data in a database.


## General requirements for migrations

* The migration SHOULD NOT break backward compatibility.
* The migration must be idempotent.
* The migration SHOULD NOT degrade the database or impact service performance.
* The migration SHOULD NOT contain irreversible operations.
* All migrations must be submitted for SQL Advocates review.


## The migration must not break backward compatibility

Deployment of an application is not instantaneous:

- At some point in time, the database appears to have already been updated, but instances of previous versions of the application are still running in production. If the updated database is incompatible with previous versions of the application, it causes failures that continue until the new version of the application completely replaces the old one.

- There may be a situation when it is necessary to roll back a version of the application (e.g.: a failed deployment), then the database must be compatible with the previous version of the application.

**In order to prevent backward compatibility failures, a new version of the application should be deployed in several phases instead of rolling it out to production all at once.**


### Example of using multiple phases in the deployment


For example, let's take the need to rename and change the data format of a column.Then, the complete list of changes includes:

1. Renaming the column.
2. Changing the format of the data in the column.
3. Writing the data in the new format.
4. Reading data in the new format.

The deployment of this functionality can be broken down into the following phases:

#### Phase 1:

1. Add a new nullable column.
2. Do not make any changes to the existing column yet.
3. When writing, save data to both columns: existing and new, in the appropriate format for each.
4. Make no changes to the data reading logic yet - continue to read from the existing column.

#### Phase 2:

1. Fill all blank values in the new column with data from the existing column, converting it to the new format.
2. Make the new column non-nullable.
3. Switch the data reading logic to the new column.
4. For now, continue to write data to both columns: the existing column and the new column.

#### Phase 3:

1. Make the existing column nullable.
2. Stop writing data to the existing column and remove all mentions of it from the code.

#### Phase 4:

1. Delete existing column from the database.

### A general list of backward compatibility compliance guidelines

* **Deleting a column.** Deleting columns is possible, but with great care and only with renaming of column before deletion. Removing unnecessary columns should be done in a separate release and scheduled separately. Only after the current build with a new column will work successfully for 2-3 releases. 
* **Table deletion.** Deleting unnecessary tables should be done in a separate release and scheduled separately. Only after the current build will work successfully for 2-3 releases you can delete the old table. It is recommended to rename the potentially deleted table beforehand to exclude the fact of using this table.
* **Changing the data type downward (e.g. TEXT -> VARCHAR, DOUBLE -> INT).** It is not allowed to reduce the data type. Reducing the type will lead to loss of the original data. If data reduction is fundamentally important, create a new column in the table, fill it with data and switch the application to it. The same rule with the creation of a new column and filling it with data is valid for columns where you need to change the data type, and the data in the table is a huge amount and alter will take a lot of time.
* **Removing data from the table.** If "unnecessary data" does not violate the logic of the application, it is not necessary to delete it. Otherwise, by agreement with the DBA - as the deletion script can put the database with a large number of deleted rows. However, if you have a very large table, you can think about partitioning or dividing the table or database into operational and archive data.


## The migration script should be idempotent

When writing migration scripts, there are a few things to keep in mind:

* Idempotent and convergent scripts describe not the actions to be performed on an object, but the state to which the object should be brought.
* Idempotency means that if such a script is executed successfully and the object is already in the desired state, then repeated execution of the script will not change anything and will not lead to anything.
* Convergence means that if the script was not executed or failed, when it is executed again, the system will strive for the desired state.


## Migration should not degrade the database and impact service performance

Performing migration puts additional load on databases, services (depending on the migration method). This should be taken into account when preparing the migration:

* All migrations must be tested on a dev(test) environment. When performing the migration, it is necessary to estimate the load created and the execution time.
* It is necessary to estimate the amount of data that will be affected by the migration in the production environment.
* It is necessary to extrapolate the results of load measurement on the dev environment to the amount of data in the prod environment.
* If the migration involves updating, deleting, changing more than 100000 rows, it is necessary to split such migration into parts. For example, to run in a loop processing of 50000 rows with commit and restart.
* When preparing the migration, it is necessary to coordinate the method of data retrieval and the load on the source with the owner team.

## The script SHOULD NOT contain irreversible operations

After the migration is performed and the corresponding functionality is deployed, it is always possible that the state will need to be rolled back to the previous version. Therefore, each migration script must have a corresponding rollback script.

The exception is the deletion of columns and records in the last stage of migration, where it is necessary to clear obsolete values.

## Methods of performing migrations

Migrations are categorized by the method of execution into:

* SQL scripts run by the DBA.
* Go code level scripts, run either at application startup or in Kubernetes Job.

To perform data migration within a single database, the preferred option would be to prepare the corresponding SQL script. The migration will be performed by the database itself, without involving the network connection to the application.

If it is necessary to migrate data between different sources, it is necessary to use migrations at the Go-code level. The sources can be: databases, Kafka, service API, etc.

When performing migration, it is necessary to monitor the process of its execution, as well as the load on databases, services, etc:

you can use existing Grafana dashboards to monitor the load on the database and services.

To monitor the migration itself, you need to add logging to the migration code or script.

The following will describe how to run data migration (DML).


### SQL scripts run by DBAs

One of the ways to migrate data is to run a prepared SQL script by a DBA specialist. The general algorithm for performing the migration in this approach is as follows:

1. The developer(s) prepares one or more *.sql scripts for migration.
2. The script is submitted to the SQL Advocates for review. If comments appear, the script is finalized. Also, migrations performed in the monolith base must pass the mandatory architects' review.
3. The script is passed to the DBA for execution.

When performing a release and migrations in several phases, linking the migration to the release is not required. A new version release and migration execution can be performed at different times.

A script can contain either a SQL query for a simple migration or a set of procedures for a more complex migration involving data conversion and loops.

Any entities created by the script (procedures, temporary tables, etc.) must be deleted after the migration is performed.


### Go-code level migrations

A Go-code level script can also be prepared for data migration. To perform such a migration, the appropriate Go-code is prepared. The requirements for Go-code level migrations are as follows:
1. The migration must necessarily be validated by the data source team at the time of load of the source service or database before running.
2. The migration must necessarily be validated by SQL Advocate.
3. No existing production code of the application may be used to perform the migration. The code for migration must be written separately.
4. After successful migration, the corresponding code must be removed.
5. The migration code should not use parallelization, but should be executed in a single thread to avoid "race" problems and excessive uncontrolled load.
6. The migration code should write logs about the progress of the execution process (migration started, migrated so much data, migration finished successfully/error).
7. The migration should have a controlled load on the database and data sources. For this purpose, appropriate rate limiters algorithms can be applied.

The following options are available for launching such migration:
1. Starting the migration at application startup.
2. Starting migration at http endpoint call

#### Starting the migration on application startup
This method is **unacceptable**, because once the migration is started, we cannot control it. If there is a need to stop the migration, we can do it only by redeploying.

#### Starting migration when http endpoint is invoked
This method of execution is similar to starting migration on application startup, except that the migration starts when the corresponding http request is received.

This run method should have two modes:

- dry-run - the script takes data but does not write it. This mode is needed to check the performance/access to data sources.

- run - the script takes data and performs migrations.

The migration is initially run in dry-run mode. At this moment the load is checked. After that a decision is made to start the migration in run mode.

Also, this way of running should have a separate endpoint to stop the migration.

## Security requirements for migrations
The following security requirements apply to migrations:

All migrations must be appropriately reviewed:

- SQL Advocates review for SQL scripts;

- a data source team review for Go-code level migrations.

The data source should be read-only resources:

- database replica;

- get API request.

Temporary secrets should be issued to access resources external to the team (other database).

## General process of migration execution
### Before performing the migration
1. It is necessary to determine the type of migration required: Liquibase script, SQL script, code-level migration. The basis for selecting the type of migration is the task that is being migrated:
    * data schema change (DDL) - Liquibase script
    * update/deletion/migration of data within one database - SQL migration
    * update/delete/move data between different sources - code level migration

2. You need to determine the amount of data/records affected by the migration in the production environment. It is necessary to extrapolate the results of load measurement in dev environment to the amount of data in prod environment.

3. It is necessary to determine the blast radius of migration.

4. Prepare migration script.

5. Check the migration execution on dev(test) environment for the moment of created load and execution time.

6. The migration script must be reviewed by the team. The following guidelines should be followed when performing the review:
    - The migration should not break backward compatibility.
    - The migration must be idempotent.
    - The migration should not contain irreversible operations.
    - The migration should not degrade the database and impact the service.
7. The migration must be coordinated with the data source owner team.
8. The migration must pass a SQL Advocates review and/or a data source team review.
9. Based on the blast radius value, make a list of services, teams, etc. that should be notified during the migration for possible failures.


### During migration execution
1. Notify the services, commands defined in the pre-stage before the migration is executed.
2. Run the migration in dry-run mode (if this mode is supported). Check the load created by the migration.
3. Start the migration in run mode.


### After migration execution
1. You need to verify that the migration was performed correctly based on logs and metrics. For example:
2. Whether the migration resulted in degradation of services.
3. Whether the data was migrated/changed in full.

Migration checklist
1. [ ] The method of migration execution has been selected
2. [ ] Migration blast radius defined
3. [ ] Migration prepared
4. [ ] Migration execution tested on dev (test) environment
5. [ ] Migration script has been reviewed by the team
6. [ ] Migration execution is agreed with the data source team
7. [ ] Migration has passed SQL Advocate review and/or data source team review
8. [ ] List of services, commands falling within blast radius defined
9. [ ] Services, commands that fall within blast radius are notified about the start of migration execution
10. [ ] Migration started in dry-run mode (if this mode is available)
11. [ ] Migration started in run mode
12. [ ] The migration has been checked for correctness