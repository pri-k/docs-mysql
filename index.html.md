---
title: MySQL for Pivotal CF
---

This is documentation for the MySQL service for Pivotal CF.

### Install via Pivotal Operations Manager

To install MySQL for Pivotal CF, follow the procedure for installing Pivotal Ops Manager tiles:

1. Download the product file from [Pivotal Network](https://network.gopivotal.com/).
1. Upload the product file to your Ops Manager installation.
1. Click **Add** next to the uploaded product description in the Available Products view
   to add this product to your staging area.
1. Click the newly added tile to review any configurable options.
1. Click **Apply Changes** to install the service.

This product requires Pivotal CF version 1.2 or greater.

### Provisioning and Binding via Cloud Foundry

Once you have installed the product, it automatically registers itself with your Elastic Runtime. At this point, the product is available to your application developers, either in the Marketplace in the web based console, or via `cf marketplace`. They can add, provision, and bind the service to their applications like any other CF service:

```
$ cf create-service p-mysql 100mb-dev mydb
$ cf bind-service myapp mydb
$ cf restart myapp
```

### Example Application

To help your application developers get started with MySQL for Pivotal CF, we have provided an example application, which can be [downloaded here][example-app].

[example-app]:mysql-example-app.tgz

### Available Plans

A single service plan enforces quotas of 100MB of storage per database and 40 concurrent connections per user. Users of Operations Manager can configure these plan quotas, as well as the maximum number of databases permitted by the server. The name of the plan is **100mb-dev** by default and is automatically updated if the storage quota is modified.

Provisioning a service instance from this plan creates a MySQL database on a multi-tenant server, suitable for development workloads. Binding applications to the instance creates unique credentials for each application to access the database.

### Service Dashboard

Cloud Foundry users can access a service dashboard for each database from Developer Console via SSO. The dashboard displays current storage utilization and plan quota. On the Space page in Developer Console, users with the SpaceDeveloper role will find a "Manage" link next to the instance. Clicking this link will log users into the service dashboard via SSO. 

### Version

Version 1.2 of this product is based on MySQL version 5.6.13.

### Known Limitations

The MySQL server runs on a single VM. The server is not replicated or redundant. Data durability depends on the infrastructure persistance layer.

### Further Reading

* [Back Up MySQL for Pivotal CF](backup.html)
* [Official MySQL Documentation](http://dev.mysql.com/doc/refman/5.6/en/)

