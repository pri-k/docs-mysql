---
title: MySQL for Pivotal CF
---

This is documentation for the MySQL service for Pivotal CF.

## Release Notes

Consult the [Release Notes](release-notes.html) for important tips and information about changes for this product.

## Known Issues

- The service dashboard is incompatible from v1.2 is incompatible with Elastic Runtime v1.3. If you have upgraded Elastic Runtime to v1.3, please install MySQL for Pivotal CF v1.3 to restore dashboard functionality.

**Note**: The MySQL server runs on a single VM. The server is not replicated or redundant. Data durability depends on the infrastructure persistence layer. This product is recommended for test and development workloads only; not recommended for production use.

## Deploy via Pivotal Operations Manager

1. Download the product file from [Pivotal Network](https://network.gopivotal.com/products/p-mysql).
1. Upload the product file to your Ops Manager installation.
1. Click **Add** next to the uploaded product description in the Available Products view
   to add this product to your staging area.
1. Click the newly added tile to review any configurable options.
1. Click **Apply Changes** to deploy the service.

This product requires Pivotal CF version 1.2 or greater.

## Configuring Lifecycle Errands

Two lifecycle errands are run by default: the **broker registrar** and the **smoke test**. The broker registrar errand registers the broker with the Cloud Controller and makes the configured plan public. The smoke test errand runs basic tests to validate that Elastic Runtime applications can successfully create, bind, use, and unbind MySQL service instances. Both errands can be turned on or off under **Lifecycle Errands** on the **Settings** tab.

## Provisioning and Binding via Cloud Foundry

Once you have installed the product it automatically registers itself with your Elastic Runtime. At this point the product is available to your application developers in the Services Marketplace, via the web-based Developer Console or `cf marketplace`. They can add, provision, and bind the service to their applications like any other CF service:

<pre class="terminal">
$ cf create-service p-mysql 100mb-dev mydb
$ cf bind-service myapp mydb
$ cf restart myapp
</pre>

## Example Application

To help your application developers get started with MySQL for Pivotal CF, we have provided an example application, which can be [downloaded here][example-app].

[example-app]:mysql-example-app.tgz

## Product Configuration

### Service Plans

A single service plan enforces quotas of 100MB of storage per database and 40 concurrent connections per user. Users of Operations Manager can configure these plan quotas. Changes to quotas will apply to all existing database instances as well as new instances. In calculating storage utilization, indexes are included along with raw tabular data.

The name of the plan is **100mb-dev** by default and is automatically updated if the storage quota is modified.

Provisioning a service instance from this plan creates a MySQL database on a multi-tenant server, suitable for development workloads. Binding applications to the instance creates unique credentials for each application to access the database.

### Instance Capacity

An operator can configure how many database instances can be provisioned (instance capacity) by the increasing the amount of persistent disk allocated to the MySQL server nodes. This is done by modifying the Persistent Disk field on the Resource Sizes page for the product in Operations Manager. Not all persistent disk will be available for instance capacity; about 2-3 GB is reserved for service operation.

## Service Dashboard

Cloud Foundry users can access a service dashboard for each database from Developer Console via SSO. The dashboard displays current storage utilization and plan quota. On the Space page in Developer Console, users with the SpaceDeveloper role will find a **Manage** link next to the instance. Clicking this link will log users into the service dashboard via SSO.

## Version

Version 1.3 of this product is based on [MariaDB](https://mariadb.org/en/) 10.0.12.

## Back Ups

See [Back Up MySQL for Pivotal CF](backup.html).

**Note**: For information about backing up your Pivotal CF installation, refer to the [Backing Up Pivotal CF](http://docs.gopivotal.com/pivotalcf/customizing/backup-settings.html) topic.
