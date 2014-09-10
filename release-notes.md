---
title: Release Notes
---

## 1.3.0

### Changes since 1.2.0
- **Syslog forwarding**: Syslogs are now streamed to the same host and port configured in Elastic Runtime settings
- **Dynamic instance capacity management**: Previously operators had to manually configure the maximum number of service instances permitted by the server. This required manual calculation and a knowledge of required system headroom. Admins can now manage instance capacity simply by adjusting persistent disk allocated to mysql nodes. Remaining instance capacity is determined dynamically by subtracting a safe estimate for system headroom and reserved storage for provisioned instances.
- **Trusty stemcells**: Server and broker are now deployed on Ubuntu “Trusty” 14.04 LTS stemcells, providing improved security, performance, and a smaller resource footprint.
- **Least necessary privileges**: The MySQL service dashboard uses a new, limited permission OAuth scope to determine whether a user currently has access to a service instance. The dashboard no longer has full read access to a user’s account.

### Known Issues

- Service is run on one multi-tenant server node. Replication is not yet supported.

## 1.2.0

### Changes since 1.1.0

* Service renamed to 'MySQL for Pivotal CF'
* Plan attributes are configurable
    * Max Storage per database
    * Max Concurrent Connections per user
    * Max databases
* Plan name is determined dynamically based on configured storage quota
* Plan features include disclaimer that the service is not for production use
* API changes:
    * Dashboard url returned on instance creation.
* New UI for Service Instance Dashboard:
    * Styled dashboard shows instance storage usage.
    * Includes warning when quota has been surpassed.
    * Uses new SSO feature in Cloud Controller API. User must approve `openid` and `cloud_controller.read` scopes for UI to work. (Scope approvals can be updated at any time.)
* Security Upgrades:
    * Updated Rails.
    * Prevention of SQL injection.
* Broker is available via URL (rather than by IP). Typically has the format `https://p-mysql.<cf-domain>`.
* Supports clustered NATS.
* Displayed plan name is configurable at deploy time.
* Bosh errands can be used to register and deregister the broker, and to run tests that verify the deployment.
* Improved logging in service broker

### The following components will be re-deployed:

* cf-mysql-broker
* mysql

### The release includes new components (all used to run Bosh errands):

* broker-registrar
* broker-deregistrar
* acceptance-tests

### Known issues

- During SSO flow, a user is required to approve the `openid` and `cloud_controller.read` scopes for the broker so that dashboard page can display the appropriate data. Subsequently revoking those approvals will not take immediate effect; there can be a delay of up to ten minutes.

## 1.1.0

### Changes since 1.0.0

* Updated the format of metadata fields in the broker catalog endpoint and added
additional fields. For more information, see Catalog Metadata.
* Updated Ruby to version 2.0.0p353 to fix a vulnerability in 1.9.3p448.
* Requests to delete a service instance or binding now get a 200 response with an empty
JSON body instead of a 204.
* The broker now returns a clear error when there is no more capacity for additional
instances during a provision request. The response has status code `507`. The
user-facing error message is "Service plan capacity has been reached."

### The following components will be re-deployed:
* cf-mysql-broker
* mysql
