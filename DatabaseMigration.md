# Database Migration in Kubernetes
What makes database management complicated is that we must preserve the data while making changes to the schema. We can’t replace the database with each release as we do with the application.

This problem is even more challenging when we consider that the database must remain online during migrations, and nothing can be lost in the event of a rollback.

### Commit database scripts to version control
Generally, there are two kinds of database scripts: data-definition language (DDL) and data-manipulation language (DML). DDL creates and modifies database structures such as tables, indexes, triggers, stored procedures, permissions, or views. DML is used to manipulate the actual data in the tables.

Like all code, both kinds of scripts should be kept in version control.

### Use database management tools
There are many tools for writing and maintaining migration scripts like Flyway, DBDeploy, Golang-migrate.The aim of all these tools is to maintain an uninterrupted set of delta scripts that upgrade and downgrade the database schema as needed.

### Decouple app deployment from data migrations
Application deployment and data migration have very different characteristics. While a deployment usually takes seconds and can occur several times a day, database migrations are more infrequent and executed outside peak hours.

We must separate data migration from application deployment since they need different approaches. Decoupling makes both tasks easier and safer.

### Rollback with CI/CD
Be it to downgrade the application or because a migration failed, there are some situations in which we have to undo database changes, effectively rolling it back to a past schema version. This is not a big problem as long as we have the rollback script and have kept changes non-destructive.

## Design approach 
The database is hosted on kubernetes as statefulsets. We can leverage the Helm charts or the Blue-green deployment approach. 

## Helm hooks
Helm provides a feature called chart hooks,which allows to execute scripts or commands at specific point during the lifecycle of kubernetes object created from helm chart.
The database build and deploy process is already handled by helm charts. Here, the trick is to use the database migration scripts as docker container image. Along with the application code, the migrations are all considered in a single helm package.The migration scripts are deployed as a kubernetes job. The Helm Hooks triggers the kubernetes Job "database migration" during the deployment process before rolling out the application.

```
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migrate
  annotations:
    # This is what defines this resource as a hook. Without this line, the
    # job is considered part of the release.
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
      - name: migrate
        image: migration-script:latest
        command: ['/app/migrate']
```

![Helmhook](https://github.com/user-attachments/assets/9b77f232-39bf-45f1-85fb-00d484305d9a)

 Advantages/ Disadvantages:
- Doesnt need extra resources.
- Modularity and reusabilty.
- Rollback is not straight forward.
- Needs some downtime.

## Blue-green deployment 
This approach uses two parallel infrastructure behind a load balancer. Blue is the stable version and Green is the inactive version. Before deployment, the inactive green system receives a current database restore from blue, and it’s kept in sync with a mirroring mechanism. Then, it is migrated to the next version.
Once the inactive system is upgraded and tested, users are switched over.

![BluegreenDeployment](https://github.com/user-attachments/assets/27075c51-e61a-4a8f-ac28-facc1b0cad2a)

 Advantages/ Disadvantages:
- Rollback is easy. The only catch with this setup is that transactions executed by the users on the green side must be replayed on blue after the rollback.
- Zero downtime
- Uses double the resources.
- Traffic switch has to be managed efficiently.
  


