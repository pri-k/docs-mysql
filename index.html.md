---
title: MySQL for Pivotal CF
---

This is documentation for the MySQL service for Pivotal CF.

### Release Notes

Find release notes at [MySQL for Pivotal CF](/pivotalcf/pcf-release-notes/p1-v1.2/mysqldev_rn.html).

### Deploy via Pivotal Operations Manager

To deploy MySQL for Pivotal CF, follow the procedure for deploying Pivotal Ops Manager tiles:

1. Download the product file from [Pivotal Network](https://network.gopivotal.com/).
1. Upload the product file to your Ops Manager installation.
1. Click **Add** next to the uploaded product description in the Available Products view
   to add this product to your staging area.
1. Click the newly added tile to review any configurable options.
1. Click **Apply Changes** to deploy the service.

This product requires Pivotal CF version 1.2 or greater.

### Configuring Lifecycle Errands

Two lifecycle errands are run be default, the broker registrar and smoke tests.  The broker registrar errand registers the broker with Cloud Controller and makes the configured plan public.  The smoke test errand runs basic tests to validate Elastic Runtime apps can successfully create, bind, use, and unbind MySQL service instances.  Both can be turned on or off in the "Lifecycle Errand" tab on the left.

### Provisioning and Binding via Cloud Foundry

Deploying the service automatically registers it with your Elastic Runtime. Once the service is deployed, it is available to your application developers from the Services Marketplace, either in the web-based Developer Console or via CLI (`cf marketplace`). Developers can create instances and bind them to their applications like any other CF service:

```
$ cf create-service p-mysql 100mb-dev mydb
$ cf bind-service myapp mydb
$ cf restart myapp
```

### Example Application

To help you get started with MySQL for Pivotal CF, we have provided an example application, which can be [downloaded here][example-app]. You'll find instructions for using the app in the included README.

[example-app]:mysql-example-app.tgz

### Service Plans

A single service plan is supported. Provisioning a service instance from this plan creates a MySQL database on a multi-tenant server, suitable for development workloads. These databases should not be used for production workloads. Binding applications to the instance creates unique credentials for each bound application to access the database.

The plan enforces default quotas of 100MB of storage per database and 40 concurrent connections per user. Users of Operations Manager can configure these quotas, as well as the maximum number of databases permitted by the server. In calculating storage utilization, indexes are included along with raw tabular data. Changes to quotas will apply to all existing database instances as well as new instances. Operators should be aware of the relationship between persistent disk, database storage quota, and max databases per server. Storage quota multiplied by max databases must be less than persistent disk, or a server could run out of disk and crash.  

The plan name is **100mb-dev** by default and is automatically updated if the storage quota is modified in Operations Manager (eg. a storage quota of 1000 would result in a plan name of 1000mb-dev).

### Service Dashboard

Cloud Foundry users can access a service dashboard for each database from Developer Console via SSO. The dashboard displays current storage utilization and plan quota. On the Space page in Developer Console, users with the SpaceDeveloper role will find a "Manage" link next to the instance. Clicking this link will log users into the service dashboard via SSO. 

### Version

Version 1.2 of this product is based on MySQL version 5.6.13.

### Known Limitations

The MySQL server runs on a single VM. The server is not replicated or redundant. Data durability depends on the infrastructure persistence layer.

### Further Reading

* [Back Up MySQL for Pivotal CF](backup.html)
* [Official MySQL Documentation](http://dev.mysql.com/doc/refman/5.6/en/)

