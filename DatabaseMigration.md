# Database Migration in Kubernetes


What makes database management complicated is that we must preserve the data while making changes to the schema. We can’t replace the database with each release as we do with the application.

This problem is even more challenging when we consider that the database must remain online during migrations, and nothing can be lost in the event of a rollback.



### Commit database scripts to version control
Generally, there are two kinds of database scripts: data-definition language (DDL) and data-manipulation language (DML). DDL creates and modifies database structures such as tables, indexes, triggers, stored procedures, permissions, or views. DML is used to manipulate the actual data in the tables.

Like all code, both kinds of scripts should be kept in version control.

### Use database management tools
There are many tools for writing and maintaining migration scripts like Flyway, DBDeploy, Go-migrate.The aim of all these tools is to maintain an uninterrupted set of delta scripts that upgrade and downgrade the database schema as needed.

### Decouple app deployment from data migrations
Application deployment and data migration have very different characteristics. While a deployment usually takes seconds and can occur several times a day, database migrations are more infrequent and executed outside peak hours.

We must separate data migration from application deployment since they need different approaches. Decoupling makes both tasks easier and safer.

### Rollback with CI/CD
Be it to downgrade the application or because a migration failed, there are some situations in which we have to undo database changes, effectively rolling it back to a past schema version. This is not a big problem as long as we have the rollback script and have kept changes non-destructive.

## Design approach

Since the database is hosted on kubernetes as a statefulset. The database build and deploy process is already handled by helm charts. So, the migrations scripts will also be build as docker container image. Helm provides a feature called chart hooks,which allows to execute scripts or commands at specific point during the lifecycle of kubernetes object created from helm chart. 

In this approach, the SQL migration scripts are packaged as a docker image along with the application code. Everything is packaged in a single helm hart. The migration scripts are deployed as a kubernetes job. 
Since the database is hosted on kubernetes as a StatefulSet, there is already an image with the database. Now there could be scenario when the schema got changed and we have to ensure the data is not lost due to schema changes. There are couple of tools for database migrations, I will be using migrate utilty from golang. 

## Design approach 

- Rollback plan - Rollbacks can be tricky due to irreversible destructive schema changes or differences in application and database versions Implement and test rollback before each migration.

- Use Helm Hooks - Create a kubernetes Job that runs your the customs migration scripts and the annotation helps trigger that job during the deployment process before rolling out the application.
 
- Monitoring and logging - 
