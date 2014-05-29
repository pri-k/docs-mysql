---
title: Back Up MySQL for Pivotal CF
---
Complete the following steps to back up Pivotal MySQL Dev.

**Note**: For information about backing up your Pivotal CF installation, refer to the [Backing Up Pivotal CF](http://docs.gopivotal.com/pivotalcf/customizing/backup-settings.html) topic.

1. Locate the name of the current BOSH deployment for Pivotal MySQL Dev:

    ```
    $bosh deployments
    +------------------------------+--------------+-------------------------------+
    | Name                         | Release(s)   | Stemcell(s)                   |
    +------------------------------+--------------+-------------------------------+
    | cf-3931915d61abfc6d8ced      | cf/155.2-dev | bosh-vsphere-esxi-ubuntu/1471 |
    +------------------------------+--------------+-------------------------------+
    | p-mysql-1e136692e37b3ec1f942 | cf-mysql/5   | bosh-vsphere-esxi-ubuntu/1471 |
    +------------------------------+--------------+-------------------------------+

    Deployments total: 2
    ```

1. Download and save the BOSH deployment manifest for Pivotal MySQL Dev. You need this manifest to locate information about your databases. Use the name you discovered in the previous step.

    ```
    $bosh download manifest p-mysql-1e136692e37b3ec1f942 mysql.yml
    Deployment manifest saved to `mysql.yml'
    ```

1. In `mysql.yml`, locate the IP address and password for the MySQL node:

    ```
    mysql_node:
      host: 172.16.78.60
      admin_password: c0eeec42e465428312fd
    services:
    - name: p-mysql
    ```

1.  Dump the data from the p-mysql DB and save it:

    `$ mysqldump -u root -p -h 172.16.78.60 --all-databases > user_databases.sql`
