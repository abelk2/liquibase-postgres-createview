# Demonstration of Liquibase bug with PostgreSQL + createView

This is a small demo of bug caused by https://liquibase.jira.com/browse/CORE-2377

## Prerequisites
* Latest PostgreSQL installed
* Databases with names `example1` and `example2` exist

## Test changelogs

### Test case #1

Database view cannot be replaced if field order is different (analogous to field renaming).

Run with `./gradlew update -PdbUsername=<user> -PdbPassword=<password> -PchangeLogs=changeFieldOrder`


```xml
<changeSet id="1" author="abelk">
    <comment>Create example view</comment>

    <createTable tableName="example_table">
        <column name="field1" type="int" />
        <column name="field2" type="int" />
    </createTable>

    <createView viewName="example_view" replaceIfExists="true">
        SELECT     field1, field2
        FROM       example_table
    </createView>
</changeSet>

<changeSet id="2" author="abelk">
    <comment>Change field order</comment>

    <createView viewName="example_view" replaceIfExists="true">
        SELECT     field2, field1
        FROM       example_table
    </createView>
</changeSet>

```

<details>
  <summary>Expand exception stack trace</summary>

  ```
  liquibase-plugin: Running the 'changeFieldOrder' activity...
10:03:41.626 INFO  [liquibase.integration.commandline.Main]: Starting Liquibase at Sze, 18 m▒rc. 2020 10:03:41 CET (version 3.8.1 #9 built at Wed Nov 06 04:48:03 UTC 2019)
10:03:42.048 INFO  [liquibase.integration.commandline.Main]: No Liquibase Pro license key supplied. Please set liquibaseProLicenseKey on command line or in liquibase.properties to use Liquibase Pro features.
10:03:42.048 INFO  [liquibase.integration.commandline.Main]: Liquibase Community 3.8.1 by Datical
10:03:42.228 INFO  [liquibase.executor.jvm.JdbcExecutor]: SELECT COUNT(*) FROM public.databasechangeloglock
10:03:42.255 INFO  [liquibase.executor.jvm.JdbcExecutor]: CREATE TABLE public.databasechangeloglock (ID INTEGER NOT NULL, LOCKED BOOLEAN NOT NULL, LOCKGRANTED TIMESTAMP WITHOUT TIME ZONE, LOCKEDBY VARCHAR(255), CONSTRAINT DATABASECHANGELOGLOCK_PKEY PRIMARY KEY (ID))
10:03:42.286 INFO  [liquibase.executor.jvm.JdbcExecutor]: SELECT COUNT(*) FROM public.databasechangeloglock
10:03:42.286 INFO  [liquibase.executor.jvm.JdbcExecutor]: DELETE FROM public.databasechangeloglock
10:03:42.286 INFO  [liquibase.executor.jvm.JdbcExecutor]: INSERT INTO public.databasechangeloglock (ID, LOCKED) VALUES (1, FALSE)
10:03:42.286 INFO  [liquibase.executor.jvm.JdbcExecutor]: SELECT LOCKED FROM public.databasechangeloglock WHERE ID=1
10:03:42.286 INFO  [liquibase.lockservice.StandardLockService]: Successfully acquired change log lock
10:03:42.286 INFO  [liquibase.servicelocator.ServiceLocator]: Can not use class liquibase.parser.core.json.JsonChangeLogParser as a Liquibase service because org.yaml.snakeyaml.constructor.BaseConstructor is not in the classpath
10:03:42.286 INFO  [liquibase.servicelocator.ServiceLocator]: Can not use class liquibase.parser.core.yaml.YamlChangeLogParser as a Liquibase service because org.yaml.snakeyaml.constructor.BaseConstructor is not in the classpath
10:03:43.567 INFO  [liquibase.changelog.StandardChangeLogHistoryService]: Creating database history table with name: public.databasechangelog
10:03:43.567 INFO  [liquibase.executor.jvm.JdbcExecutor]: CREATE TABLE public.databasechangelog (ID VARCHAR(255) NOT NULL, AUTHOR VARCHAR(255) NOT NULL, FILENAME VARCHAR(255) NOT NULL, DATEEXECUTED TIMESTAMP WITHOUT TIME ZONE NOT NULL, ORDEREXECUTED INTEGER NOT NULL, EXECTYPE VARCHAR(10) NOT NULL, MD5SUM VARCHAR(35), DESCRIPTION VARCHAR(255), COMMENTS VARCHAR(255), TAG VARCHAR(255), LIQUIBASE VARCHAR(20), CONTEXTS VARCHAR(255), LABELS VARCHAR(255), DEPLOYMENT_ID VARCHAR(10))
10:03:43.567 INFO  [liquibase.executor.jvm.JdbcExecutor]: SELECT COUNT(*) FROM public.databasechangelog
10:03:43.567 INFO  [liquibase.changelog.StandardChangeLogHistoryService]: Reading from public.databasechangelog
10:03:43.567 INFO  [liquibase.executor.jvm.JdbcExecutor]: SELECT * FROM public.databasechangelog ORDER BY DATEEXECUTED ASC, ORDEREXECUTED ASC
10:03:43.567 INFO  [liquibase.executor.jvm.JdbcExecutor]: SELECT COUNT(*) FROM public.databasechangeloglock
10:03:43.583 INFO  [liquibase.executor.jvm.JdbcExecutor]: CREATE TABLE public.example_table (field1 INTEGER, field2 INTEGER)
10:03:43.583 INFO  [liquibase.changelog.ChangeSet]: Table example_table created
10:03:43.583 INFO  [liquibase.executor.jvm.JdbcExecutor]: CREATE OR REPLACE VIEW public.example_view AS SELECT     field1, field2
          FROM       example_table
10:03:43.583 INFO  [liquibase.changelog.ChangeSet]: View example_view created
10:03:43.583 INFO  [liquibase.changelog.ChangeSet]: ChangeSet src/main/resources/changelogs/change-field-order.xml::1::abelk ran successfully in 0ms
10:03:43.583 INFO  [liquibase.executor.jvm.JdbcExecutor]: SELECT MAX(ORDEREXECUTED) FROM public.databasechangelog
10:03:43.583 INFO  [liquibase.executor.jvm.JdbcExecutor]: INSERT INTO public.databasechangelog (ID, AUTHOR, FILENAME, DATEEXECUTED, ORDEREXECUTED, MD5SUM, DESCRIPTION, COMMENTS, EXECTYPE, CONTEXTS, LABELS, LIQUIBASE, DEPLOYMENT_ID) VALUES ('1', 'abelk', 'src/main/resources/changelogs/change-field-order.xml', NOW(), 1, '8:2c20b522c43f03a1fcf3ed396a8c278d', 'createTable tableName=example_table; createView viewName=example_view', 'Create example view', 'EXECUTED', NULL, NULL, '3.8.1', '4522223567')
10:03:43.598 INFO  [liquibase.executor.jvm.JdbcExecutor]: CREATE OR REPLACE VIEW public.example_view AS SELECT     field2, field1
          FROM       example_table
10:03:43.598 ERROR [liquibase.changelog.ChangeSet]: Change Set src/main/resources/changelogs/change-field-order.xml::2::abelk failed.  Error: ERROR: cannot change name of view column "field1" to "field2" [Failed SQL: (0) CREATE OR REPLACE VIEW public.example_view AS SELECT     field2, field1
          FROM       example_table]
10:03:43.598 INFO  [liquibase.lockservice.StandardLockService]: Successfully released change log lock
10:03:43.598 ERROR [liquibase.integration.commandline.Main]: Unexpected error running Liquibase: ERROR: cannot change name of view column "field1" to "field2" [Failed SQL: (0) CREATE OR REPLACE VIEW public.example_view AS SELECT     field2, field1
          FROM       example_table]
liquibase.exception.MigrationFailedException: Migration failed for change set src/main/resources/changelogs/change-field-order.xml::2::abelk:
     Reason: liquibase.exception.DatabaseException: ERROR: cannot change name of view column "field1" to "field2" [Failed SQL: (0) CREATE OR REPLACE VIEW public.example_view AS SELECT     field2, field1
          FROM       example_table]
        at liquibase.changelog.ChangeSet.execute(ChangeSet.java:646)
        at liquibase.changelog.visitor.UpdateVisitor.visit(UpdateVisitor.java:53)
        at liquibase.changelog.ChangeLogIterator.run(ChangeLogIterator.java:83)
        at liquibase.Liquibase.update(Liquibase.java:202)
        at liquibase.Liquibase.update(Liquibase.java:179)
        at liquibase.integration.commandline.Main.doMigration(Main.java:1399)
        at liquibase.integration.commandline.Main.run(Main.java:229)
        at liquibase.integration.commandline.Main.main(Main.java:143)
Caused by: liquibase.exception.DatabaseException: ERROR: cannot change name of view column "field1" to "field2" [Failed SQL: (0) CREATE OR REPLACE VIEW public.example_view AS SELECT     field2, field1
          FROM       example_table]
        at liquibase.executor.jvm.JdbcExecutor$ExecuteStatementCallback.doInStatement(JdbcExecutor.java:402)
        at liquibase.executor.jvm.JdbcExecutor.execute(JdbcExecutor.java:59)
        at liquibase.executor.jvm.JdbcExecutor.execute(JdbcExecutor.java:131)
        at liquibase.database.AbstractJdbcDatabase.execute(AbstractJdbcDatabase.java:1274)
        at liquibase.database.AbstractJdbcDatabase.executeStatements(AbstractJdbcDatabase.java:1256)
        at liquibase.changelog.ChangeSet.execute(ChangeSet.java:609)
        ... 7 common frames omitted
Caused by: org.postgresql.util.PSQLException: ERROR: cannot change name of view column "field1" to "field2"
        at org.postgresql.core.v3.QueryExecutorImpl.receiveErrorResponse(QueryExecutorImpl.java:2578)
        at org.postgresql.core.v3.QueryExecutorImpl.processResults(QueryExecutorImpl.java:2313)
        at org.postgresql.core.v3.QueryExecutorImpl.execute(QueryExecutorImpl.java:331)
        at org.postgresql.jdbc.PgStatement.executeInternal(PgStatement.java:448)
        at org.postgresql.jdbc.PgStatement.execute(PgStatement.java:369)
        at org.postgresql.jdbc.PgStatement.executeWithFlags(PgStatement.java:310)
        at org.postgresql.jdbc.PgStatement.executeCachedSql(PgStatement.java:296)
        at org.postgresql.jdbc.PgStatement.executeWithFlags(PgStatement.java:273)
        at org.postgresql.jdbc.PgStatement.execute(PgStatement.java:268)
        at liquibase.executor.jvm.JdbcExecutor$ExecuteStatementCallback.doInStatement(JdbcExecutor.java:398)
        ... 12 common frames omitted
  ```
</details>

### Test case #2

Database view cannot be replaced if field data types are different.

Run with `./gradlew update -PdbUsername=<user> -PdbPassword=<password> -PchangeLogs=changeFieldDataType`

```xml
<changeSet id="1" author="abelk">
    <comment>Create example view</comment>

    <createTable tableName="example_table">
        <column name="field1" type="int" />
        <column name="field2" type="int" />
    </createTable>

    <createView viewName="example_view" replaceIfExists="true">
        SELECT     field1, field2
        FROM       example_table
    </createView>
</changeSet>

<changeSet id="2" author="abelk">
    <comment>Change field datatype</comment>

    <createView viewName="example_view" replaceIfExists="true">
        SELECT     field1, field2::numeric
        FROM       example_table
    </createView>
</changeSet>
```

<details>
  <summary>Expand exception stack trace</summary>

  ```
liquibase-plugin: Running the 'changeFieldDataType' activity...
10:44:54.559 INFO  [liquibase.integration.commandline.Main]: Starting Liquibase at Sze, 18 m▒rc. 2020 10:44:54 CET (version 3.8.1 #9 built at Wed Nov 06 04:48:03 UTC 2019)
10:44:54.998 INFO  [liquibase.integration.commandline.Main]: No Liquibase Pro license key supplied. Please set liquibaseProLicenseKey on command line or in liquibase.properties to use Liquibase Pro features.
10:44:54.998 INFO  [liquibase.integration.commandline.Main]: Liquibase Community 3.8.1 by Datical
10:44:55.201 INFO  [liquibase.executor.jvm.JdbcExecutor]: SELECT COUNT(*) FROM public.databasechangeloglock
10:44:55.214 INFO  [liquibase.executor.jvm.JdbcExecutor]: CREATE TABLE public.databasechangeloglock (ID INTEGER NOT NULL, LOCKED BOOLEAN NOT NULL, LOCKGRANTED TIMESTAMP WITHOUT TIME ZONE, LOCKEDBY VARCHAR(255), CONSTRAINT DATABASECHANGELOGLOCK_PKEY PRIMARY KEY (ID))
10:44:55.232 INFO  [liquibase.executor.jvm.JdbcExecutor]: SELECT COUNT(*) FROM public.databasechangeloglock
10:44:55.234 INFO  [liquibase.executor.jvm.JdbcExecutor]: DELETE FROM public.databasechangeloglock
10:44:55.235 INFO  [liquibase.executor.jvm.JdbcExecutor]: INSERT INTO public.databasechangeloglock (ID, LOCKED) VALUES (1, FALSE)
10:44:55.236 INFO  [liquibase.executor.jvm.JdbcExecutor]: SELECT LOCKED FROM public.databasechangeloglock WHERE ID=1
10:44:55.240 INFO  [liquibase.lockservice.StandardLockService]: Successfully acquired change log lock
10:44:55.241 INFO  [liquibase.servicelocator.ServiceLocator]: Can not use class liquibase.parser.core.json.JsonChangeLogParser as a Liquibase service because org.yaml.snakeyaml.constructor.BaseConstructor is not in the classpath
10:44:55.242 INFO  [liquibase.servicelocator.ServiceLocator]: Can not use class liquibase.parser.core.yaml.YamlChangeLogParser as a Liquibase service because org.yaml.snakeyaml.constructor.BaseConstructor is not in the classpath
10:44:56.458 INFO  [liquibase.changelog.StandardChangeLogHistoryService]: Creating database history table with name: public.databasechangelog
10:44:56.460 INFO  [liquibase.executor.jvm.JdbcExecutor]: CREATE TABLE public.databasechangelog (ID VARCHAR(255) NOT NULL, AUTHOR VARCHAR(255) NOT NULL, FILENAME VARCHAR(255) NOT NULL, DATEEXECUTED TIMESTAMP WITHOUT TIME ZONE NOT NULL, ORDEREXECUTED INTEGER NOT NULL, EXECTYPE VARCHAR(10) NOT NULL, MD5SUM VARCHAR(35), DESCRIPTION VARCHAR(255), COMMENTS VARCHAR(255), TAG VARCHAR(255), LIQUIBASE VARCHAR(20), CONTEXTS VARCHAR(255), LABELS VARCHAR(255), DEPLOYMENT_ID VARCHAR(10))
10:44:56.478 INFO  [liquibase.executor.jvm.JdbcExecutor]: SELECT COUNT(*) FROM public.databasechangelog
10:44:56.479 INFO  [liquibase.changelog.StandardChangeLogHistoryService]: Reading from public.databasechangelog
10:44:56.480 INFO  [liquibase.executor.jvm.JdbcExecutor]: SELECT * FROM public.databasechangelog ORDER BY DATEEXECUTED ASC, ORDEREXECUTED ASC
10:44:56.481 INFO  [liquibase.executor.jvm.JdbcExecutor]: SELECT COUNT(*) FROM public.databasechangeloglock
10:44:56.493 INFO  [liquibase.executor.jvm.JdbcExecutor]: CREATE TABLE public.example_table (field1 INTEGER, field2 INTEGER)
10:44:56.494 INFO  [liquibase.changelog.ChangeSet]: Table example_table created
10:44:56.495 INFO  [liquibase.executor.jvm.JdbcExecutor]: CREATE OR REPLACE VIEW public.example_view AS SELECT     field1, field2
          FROM       example_table
10:44:56.497 INFO  [liquibase.changelog.ChangeSet]: View example_view created
10:44:56.502 INFO  [liquibase.changelog.ChangeSet]: ChangeSet src/main/resources/changelogs/change-field-datatype.xml::1::abelk ran successfully in 10ms
10:44:56.502 INFO  [liquibase.executor.jvm.JdbcExecutor]: SELECT MAX(ORDEREXECUTED) FROM public.databasechangelog
10:44:56.504 INFO  [liquibase.executor.jvm.JdbcExecutor]: INSERT INTO public.databasechangelog (ID, AUTHOR, FILENAME, DATEEXECUTED, ORDEREXECUTED, MD5SUM, DESCRIPTION, COMMENTS, EXECTYPE, CONTEXTS, LABELS, LIQUIBASE, DEPLOYMENT_ID) VALUES ('1', 'abelk', 'src/main/resources/changelogs/change-field-datatype.xml', NOW(), 1, '8:2c20b522c43f03a1fcf3ed396a8c278d', 'createTable tableName=example_table; createView viewName=example_view', 'Create example view', 'EXECUTED', NULL, NULL, '3.8.1', '4524696481')
10:44:56.523 INFO  [liquibase.executor.jvm.JdbcExecutor]: CREATE OR REPLACE VIEW public.example_view AS SELECT     field1, field2::numeric
          FROM       example_table
10:44:56.524 ERROR [liquibase.changelog.ChangeSet]: Change Set src/main/resources/changelogs/change-field-datatype.xml::2::abelk failed.  Error: ERROR: cannot change data type of view column "field2" from integer to numeric [Failed SQL: (0) CREATE OR REPLACE VIEW public.example_view AS SELECT     field1, field2::numeric
          FROM       example_table]
10:44:56.525 INFO  [liquibase.lockservice.StandardLockService]: Successfully released change log lock
10:44:56.526 ERROR [liquibase.integration.commandline.Main]: Unexpected error running Liquibase: ERROR: cannot change data type of view column "field2" from integer to numeric [Failed SQL: (0) CREATE OR REPLACE VIEW public.example_view AS SELECT     field1, field2::numeric
          FROM       example_table]
liquibase.exception.MigrationFailedException: Migration failed for change set src/main/resources/changelogs/change-field-datatype.xml::2::abelk:
     Reason: liquibase.exception.DatabaseException: ERROR: cannot change data type of view column "field2" from integer to numeric [Failed SQL: (0) CREATE OR REPLACE VIEW public.example_view AS SELECT     field1, field2::numeric
          FROM       example_table]
        at liquibase.changelog.ChangeSet.execute(ChangeSet.java:646)
        at liquibase.changelog.visitor.UpdateVisitor.visit(UpdateVisitor.java:53)
        at liquibase.changelog.ChangeLogIterator.run(ChangeLogIterator.java:83)
        at liquibase.Liquibase.update(Liquibase.java:202)
        at liquibase.Liquibase.update(Liquibase.java:179)
        at liquibase.integration.commandline.Main.doMigration(Main.java:1399)
        at liquibase.integration.commandline.Main.run(Main.java:229)
        at liquibase.integration.commandline.Main.main(Main.java:143)
Caused by: liquibase.exception.DatabaseException: ERROR: cannot change data type of view column "field2" from integer to numeric [Failed SQL: (0) CREATE OR REPLACE VIEW public.example_view AS SELECT     field1, field2::numeric
          FROM       example_table]
        at liquibase.executor.jvm.JdbcExecutor$ExecuteStatementCallback.doInStatement(JdbcExecutor.java:402)
        at liquibase.executor.jvm.JdbcExecutor.execute(JdbcExecutor.java:59)
        at liquibase.executor.jvm.JdbcExecutor.execute(JdbcExecutor.java:131)
        at liquibase.database.AbstractJdbcDatabase.execute(AbstractJdbcDatabase.java:1274)
        at liquibase.database.AbstractJdbcDatabase.executeStatements(AbstractJdbcDatabase.java:1256)
        at liquibase.changelog.ChangeSet.execute(ChangeSet.java:609)
        ... 7 common frames omitted
Caused by: org.postgresql.util.PSQLException: ERROR: cannot change data type of view column "field2" from integer to numeric
        at org.postgresql.core.v3.QueryExecutorImpl.receiveErrorResponse(QueryExecutorImpl.java:2578)
        at org.postgresql.core.v3.QueryExecutorImpl.processResults(QueryExecutorImpl.java:2313)
        at org.postgresql.core.v3.QueryExecutorImpl.execute(QueryExecutorImpl.java:331)
        at org.postgresql.jdbc.PgStatement.executeInternal(PgStatement.java:448)
        at org.postgresql.jdbc.PgStatement.execute(PgStatement.java:369)
        at org.postgresql.jdbc.PgStatement.executeWithFlags(PgStatement.java:310)
        at org.postgresql.jdbc.PgStatement.executeCachedSql(PgStatement.java:296)
        at org.postgresql.jdbc.PgStatement.executeWithFlags(PgStatement.java:273)
        at org.postgresql.jdbc.PgStatement.execute(PgStatement.java:268)
        at liquibase.executor.jvm.JdbcExecutor$ExecuteStatementCallback.doInStatement(JdbcExecutor.java:398)
        ... 12 common frames omitted
  ```
</details>