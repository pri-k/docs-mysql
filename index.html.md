---
title: MySQL for Pivotal CF
---

This is documentation for the MySQL service for Pivotal CF.

## <a id="release-notes"></a>Release Notes ##

Consult the [Release Notes](release-notes.html) for information about changes between versions of this product.

## <a id="limitations"></a>Limitations ##

- **This product is recommended for test and development workloads only; it is not recommended for production use.**
- Only the InnoDB storage engine is supported; it is the default storage engine for new tables. Attempted use of other storage engines (including MyISAM) may result in data loss.
- All databases are managed by shared, multi-tenant server processes. Although data is securely isolated between tenants using unique credentials, application performance may be impacted by noisy neighbors.
- [MariaDB Galera Cluster - Known Limitations](https://mariadb.com/kb/en/mariadb/mariadb-galera-cluster-known-limitations/).

## <a id="known-issues"></a>Known Issues ##

- If a cluster node becomes a member of a partitioned minority of cluster nodes, proxy instances will route new connections to a healthy cluster node. Existing client connections will see writes rejected but connections will not be severed, and may have to wait for a timeout before they can reconnect.  
- All proxy instances use the same method to determine cluster health but during brief node failures proxy instances may come to different conclusions. Determination of which cluster node should be considered primary is not synchronized across proxy instances. If proxy instances diverge in their view of which node is primary, connections through multiple proxy instances may reach different cluster nodes. This is only an issue for tables that receive highly concurrent writes. In this scenario, multiple clients writing to the same table can obtain locks on the same row, resulting in a deadlock; one commit will succeed, all others will fail and must be retried. This can be prevented by configuring your load balancer to route connections to only one proxy instance at a time.
- Once the product is deployed with operator-configured proxy IPs, the number of proxy instances can not be reduced, nor can the configured IPs be removed from the **Proxy IPs** field. If instead the product is initially deployed without proxy IPs, IPs added to the **Proxy IPs** field will only be used when adding additional proxy instances, scaling down is unpredictably permitted, and the first proxy instance can never be assigned an operator-configured IP.

## <a id="installation"></a>Installation ##

This product requires Pivotal CF version 1.3.4 or greater.

1. Download the product file from [Pivotal Network](https://network.pivotal.io/products/p-mysql).
1. Upload the product file to your Ops Manager installation.
1. Click **Add** next to the uploaded product description in the Available Products view
   to add this product to your staging area.
1. Click the newly added tile to review configurable [Settings](#settings).
1. Click **Apply Changes** to deploy the service.

## <a id="settings"></a>Settings ##

### <a id="service-plan"></a>Service Plan ###

A single service plan enforces quotas of 100MB of storage per database and 40 concurrent connections per user by default. Users of Operations Manager can configure these plan quotas. Changes to quotas will apply to all existing database instances as well as new instances. In calculating storage utilization, indexes are included along with raw tabular data.

The name of the plan is **100mb-dev** by default and is automatically updated if the storage quota is modified.

Provisioning a service instance from this plan creates a MySQL database on a multi-tenant server, suitable for development workloads. Binding applications to the instance creates unique credentials for each application to access the database.

### <a id="proxy"></a>Proxy ###

The proxy tier is responsible for routing connections from applications to healthy MariaDB cluster nodes, even in the event of node failure.

Applications are provided with a hostname or IP address to reach a database managed by the service (for more information, see [About Service Binding](/pivotalcf/devguide/services/#binding-services)). By default, the MySQL service will provide bound applications with the IP of the first instance in the proxy tier. Even if additional proxy instances are deployed, client connections will not be routed through them. This means the first proxy instance is a single point of failure.

**In order to eliminate the first proxy instance as a single point of failure, operators must configure a load balancer to route client connections to all proxy IPs, and configure the MySQL service to give bound applications a hostname or IP address that resolves to the load balancer.**

In order to configure a load balancer with the IPs of the proxy tier before v1.4.0 is deployed and prevent applications from obtaining the IP of the first proxy instance, the product enables an operator to configure the IPs that will be assigned to proxy instances. The following instructions applies to the **Proxy** settings page for the MySQL product in Operation Manager.

1. In the **Proxy IPs** field, enter a list of IP addresses that should be assigned to the proxy instances. These IPs must be in the CIDR range configured in the Director tile and not be currently allocated to another VM. Look at the **Status** pages of other tiles to see what IPs are in use.
- Configure your load balancer to route connections for a hostname or IP to the proxy IPs. As proxy instances are not synchronized, we recommend configuring your load balancer to send all traffic to one proxy instance at a time until it fails, then failover to another proxy instance. For details, see [Known Issues](#known-issues).
- In the **Binding Credentials Hostname** field, enter the hostname or IP that should be given to bound applications for connecting to databases managed by the service. This hostname or IP should resolve to your load balancer and be considered long-lived. When this field is modified applications must be rebound to receive updated credentials.

If v1.4.0 is initially deployed without a load balancer and without proxy IPs configured, a load balancer can be setup later to remove the proxy as a single point of failure. However, there are several implications to consider:

- Applications will have to be rebound to receive the hostname or IP that resolves to the load balancer. To rebind: unbind your application from the service instance, bind it again, then restage your application. For more information see [Managing Service Instances with the CLI](/pivotalcf/devguide/services/managing-services.html). In order to avoid unnecessary rebinding, we recommend configuring a load balancer before deploying v1.4.0.
- Instead of configuring the proxy IPs in Operations manager, use the IPs that were dynamically assigned by looking at the **Status** page. Configuration of proxy IPs after the product is deployed with dynamically assigned IPs is not well supported; see [Known Issues](#known-issues).

In older versions of the product, applications were given the IP of the single MySQL server in bind credentials. When upgrading to v1.4, existing applications will continue to function but in order to take advantage of high availability features they must be rebound to receive either the IP of the first proxy instance or the IP/hostname of a load balancer.

### <a id="lifecycle-errands"></a>Lifecycle Errands ###

Two lifecycle errands are run by default: the **broker registrar** and the **smoke test**. The broker registrar errand registers the broker with the Cloud Controller and makes the service plan public. The smoke test errand runs basic tests to validate that service instances can be created and deleted, and that applications pushed to Elastic Runtime can be bound and write to MySQL service instances. Both errands can be turned on or off on the **Lifecycle Errands** page under the **Settings** tab.

### <a id="resource-config"></a>Resource Config ###

#### <a id="instance-capacity"></a>Instance Capacity ####

An operator can configure how many database instances can be provisioned (instance capacity) by configuring the amount of persistent disk allocated to the MySQL server nodes. The broker will  provision a requested database if there is sufficient unreserved persistent disk. This can be managed using the Persistent Disk field for the MySQL Server job in the Resource Config setting page in Operations Manager. Not all persistent disk will be available for instance capacity; about 2-3 GB is reserved for service operation. Adding nodes to the cluster increases durability, not capacity. Multiple backend clusters, to increase capacity or for isolation, are not yet supported.

In determining how much persistent disk to make available for databases, operators should also consider that MariaDB servers require sufficient CPU, RAM, and IOPS to promptly respond to client requests for all databases.

## <a id="provision-and-bind"></a>Provisioning and Binding via Cloud Foundry ##

As part of installation the product is automatically registered with Elastic Runtime (see [Lifecycle Errands](#lifecycle-errands)). On successful installation, the MySQL service is available to application developers in the Services Marketplace, via the web-based Developer Console or `cf marketplace`. Developers can then provision instances of the service and bind them to their applications:

<pre class="terminal">
$ cf create-service p-mysql 100mb-dev mydb
$ cf bind-service myapp mydb
$ cf restart myapp
</pre>

For more information on use of services, see the [Services Overview](/pivotalcf/devguide/services/).

## <a id="example-app"></a>Example Application ##

To help application developers get started with MySQL for Pivotal CF, we have provided an example application, which can be [downloaded here][example-app]. Instructions can be found in the included README.

[example-app]:mysql-example-app.tgz

## <a id="dashboard"></a>Service Dashboard ##

Cloud Foundry users can access a service dashboard for each database from Developer Console via SSO. The dashboard displays current storage utilization and plan quota. On the Space page in Developer Console, users with the SpaceDeveloper role will find a **Manage** link next to the instance. Clicking this link will log users into the service dashboard via SSO.

## <a id="haproxy-stats"></a>HAProxy Statistics Dashboard ##

The service provides a dashboard where administrators can observe metrics for each instance in the proxy tier. Metrics include the number of clients currently connected and the number of connections made to each of the backend database nodes.

This statistics dashboard for each proxy instance can be found at: `http://haproxy-<job index>.p-mysql.<system-domain>`. Job index starts at 0; if you have two proxy instances deployed and your system-domain is `example.com`, dashboards would be accessible at `http://haproxy-0.p-mysql.example.com` and `http://haproxy-1.p-mysql.example.com`.

Basic auth credentials are required to access the dashboard. These can be found in the Credentials tab of the MySQL product in Operations Manager.

## <a id="backup"></a>Back Ups ##

See [Back Up MySQL for Pivotal CF](backup.html).

**Note**: For information about backing up your Pivotal CF installation, refer to the [Backing Up Pivotal CF](/pivotalcf/customizing/backup-settings.html) topic.

## <a id="version"></a>Version ##

Version 1.4 of this product is based on [MariaDB Galera Cluster](https://mariadb.com/kb/en/mariadb/documentation/replication/galera/what-is-mariadb-galera-cluster/) 10.0.13.
