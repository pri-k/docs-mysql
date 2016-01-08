---
title: Bootstrapping a Galera Cluster
---


Bootstrapping is the process of (re)starting a Galera cluster. Before evaluating whether manual bootstrapping is necessary, ensure the nodes are able to communicate with each other, i.e., there are no network partitions. Once network partitions have been resolved, reevaluate the cluster state.

## When to Bootstrap

Manual bootstrapping should only be required when the cluster has lost quorum.

Quorum is lost when less than half of the nodes can communicate with each other (for longer than the configured grace period).

If quorum has *not* been lost, then individual unhealthy nodes should automatically rejoin the quorum once repaired (error resolved, node restarted, or connectivity restored).

Note: The cluster is automatically bootstrapped the first time the cluster is deployed.

### Symptoms of Lost Quorum

- All nodes appear "Unhealthy" on the proxy dashboard.
- All responsive nodes report the value of `wsrep_cluster_status` as `non-Primary`.

    ```sh
    mysql> SHOW STATUS LIKE 'wsrep_cluster_status';
    +----------------------+-------------+
    | Variable_name        | Value       |
    +----------------------+-------------+
    | wsrep_cluster_status | non-Primary |
    +----------------------+-------------+
    ```
- All responsive nodes respond with `ERROR 1047` when queried with most statement types.

    ```sh
    mysql> select * from mysql.user;
    ERROR 1047 (08S01) at line 1: WSREP has not yet prepared node for application use
    ```

See [Cluster Behavior](./cluster-behavior.html) for more details about determining cluster state.

## Bootstrapping

Bootstrapping requires you to run commands from the [Ops Manager Director](http://docs.pivotal.io/pivotalcf/customizing/index.html). Follow the instructions to use the [BOSH CLI](https://docs.pivotal.io/pivotalcf/customizing/trouble-advanced.html#prepare) for command-line access.

### Automated Bootstrap
P-mysql versions 1.8.0 and later include a [BOSH errand](http://bosh.io/docs/jobs.html#jobs-vs-errands) to automate the process of bootstrapping. In most cases, running the errand is sufficient, however there are some conditions which require additional steps.

### Scenario 1 - Virtual Machines running, Cluster Disrupted

If the nodes are up and running, but the jobs themselves are failing, the output of `bosh instances` will look like this:

   ```
   $ bosh instances
   Acting as user 'director' on deployment 'p-mysql-5c0aa34449960359a416' on 'microbosh-94ef1f7745141534babf'

   Director task 2289

   Task 2289 done

   +--------------------------------------------------+---------+------------------------------------------------+------------+
   | Instance                                         | State   | Resource Pool                                  | IPs        |
   +--------------------------------------------------+---------+------------------------------------------------+------------+
   | cf-mysql-broker-partition-a813339fde9330e9b905/0 | running | cf-mysql-broker-partition-a813339fde9330e9b905 | 10.0.16.61 |
   | cf-mysql-broker-partition-a813339fde9330e9b905/1 | running | cf-mysql-broker-partition-a813339fde9330e9b905 | 10.0.16.62 |
   | mysql-partition-a813339fde9330e9b905/0           | failing | mysql-partition-a813339fde9330e9b905           | 10.0.16.55 |
   | mysql-partition-a813339fde9330e9b905/1           | failing | mysql-partition-a813339fde9330e9b905           | 10.0.16.56 |
   | mysql-partition-a813339fde9330e9b905/2           | failing | mysql-partition-a813339fde9330e9b905           | 10.0.16.57 |
   | proxy-partition-a813339fde9330e9b905/0           | running | proxy-partition-a813339fde9330e9b905           | 10.0.16.59 |
   | proxy-partition-a813339fde9330e9b905/1           | running | proxy-partition-a813339fde9330e9b905           | 10.0.16.60 |
   +--------------------------------------------------+---------+------------------------------------------------+------------+
   ```
In this situation, it is OK to immediately try the bootstrap errand:
   > $ bosh run errand bootstrap

You will see many lines of output, eventually followed by:
   ```
   Bootstrap errand completed

   [stderr]
   + echo 'Started bootstrap errand ...'
   + JOB_DIR=/var/vcap/jobs/bootstrap
   + CONFIG_PATH=/var/vcap/jobs/bootstrap/config/config.yml
   + /var/vcap/packages/bootstrap/bin/cf-mysql-bootstrap -configPath=/var/vcap/jobs/bootstrap/config/config.yml
   + echo 'Bootstrap errand completed'
   + exit 0

   Errand `bootstrap' completed successfully (exit code 0)
   ```
There are times when this won't work immediately. Unfortunately, sometimes it is best to wait and try again a few minutes later.

### Scenario 2 - Virtual Machines Rebooted

   ```
   $ bosh instances
   +--------------------------------------------------+--------------------+------------------------------------------------+------------+
   | Instance                                         | State              | Resource Pool                                  | IPs        |
   +--------------------------------------------------+--------------------+------------------------------------------------+------------+
   | unknown/unknown                                  | unresponsive agent |                                                |            |
   +--------------------------------------------------+--------------------+------------------------------------------------+------------+
   | unknown/unknown                                  | unresponsive agent |                                                |            |
   +--------------------------------------------------+--------------------+------------------------------------------------+------------+
   | unknown/unknown                                  | unresponsive agent |                                                |            |
   +--------------------------------------------------+--------------------+------------------------------------------------+------------+
   | cf-mysql-broker-partition-e97dae91e44681e0b543/0 | running            | cf-mysql-broker-partition-e97dae91e44681e0b543 | 10.0.16.65 |
   | cf-mysql-broker-partition-e97dae91e44681e0b543/1 | running            | cf-mysql-broker-partition-e97dae91e44681e0b543 | 10.0.16.66 |
   +--------------------------------------------------+--------------------+------------------------------------------------+------------+
   | proxy-partition-e97dae91e44681e0b543/0           | running            | proxy-partition-e97dae91e44681e0b543           | 10.0.16.63 |
   | proxy-partition-e97dae91e44681e0b543/1           | running            | proxy-partition-e97dae91e44681e0b543           | 10.0.16.64 |
   +--------------------------------------------------+--------------------+------------------------------------------------+------------+
   ```

### Part 1 - VM Recovery

VM Recovery is best left to OpsManager by configuring the VM Resurrector. If so, the system will notice that the VMs are gone, and automatically attempt to recreate them. You will be able to see evidence of that by seeing the scan-and-fix job in the output of `bosh tasks recent --no-filter`:
   ```
   +-----+------------+-------------------------+----------+--------------------------------------------+---------------------------------------------------+
   | #   | State      | Timestamp               | User     | Description                                | Result                                            |
   +-----+------------+-------------------------+----------+--------------------------------------------+---------------------------------------------------+
   | 123 | queued     | 2016-01-08 00:18:07 UTC | director | scan and fix                               |                                                   |
   ```

By watching `bosh instances` you'll see the VMs transition from `unresponsive agent` to `starting` and ultimately, two will appear as `failing`:
   ```
   +--------------------------------------------------+----------+------------------------------------------------+------------+
   | mysql-partition-e97dae91e44681e0b543/0           | starting | mysql-partition-e97dae91e44681e0b543           | 10.0.16.60 |
   | mysql-partition-e97dae91e44681e0b543/1           | failing  | mysql-partition-e97dae91e44681e0b543           | 10.0.16.61 |
   | mysql-partition-e97dae91e44681e0b543/2           | failing  | mysql-partition-e97dae91e44681e0b543           | 10.0.16.62 |
   +--------------------------------------------------+----------+------------------------------------------------+------------+
   ```

### Manual Bootstrap
Once it has been determined that bootstrapping is required, follow the following steps to shut down the cluster and bootstrap from the nodes with the most transactions.

- SSH to each node in the cluster and, as root, shut down the mariadb process.

  ```sh
  $ monit stop mariadb_ctrl
  ```

  Re-bootstrapping the cluster will not be successful unless all other nodes have been shut down.

- Choose a node to bootstrap.
    - Find the node with the highest transaction sequence number (seqno).

    - If a node shutdown gracefully, the seqno should be in the galera state file.

        ```sh
        $ cat /var/vcap/store/mysql/grastate.dat | grep 'seqno:'
        ```

    - If the node crashed or was killed, the seqno in the galera state file should be `-1`. In this case, the seqno may be recoverable from the database. The following command will cause the database to start up, log the recovered sequence number, and then exit.

        ```sh
        $ /var/vcap/packages/mariadb/bin/mysqld --wsrep-recover
        ```

        Scan the error log for the recovered sequence number (the last number after the group id (uuid) is the recovered seqno):

        ```sh
        $ grep "Recovered position" /var/vcap/sys/log/mysql/mysql.err.log | tail -1
        150225 18:09:42 mysqld_safe WSREP: Recovered position e93955c7-b797-11e4-9faa-9a6f0b73eb46:15
        ```

        Note: The galera state file will still say `seqno: -1` afterward.

    - If the node never connected to the cluster before crashing, it may not even have a group id (uuid in grastate.dat). In this case, there is nothing to recover. Unless all nodes crashed this way, do not choose this node for bootstrapping.

    Use the node with the highest `seqno` value as the new bootstrap node. If all nodes have the same `seqno`, you can choose any node as the new bootstrap node.

  **Important**: Only perform these bootstrap commands on the node with the highest `seqno`. Otherwise the node with the highest `seqno` will be unable to join the new cluster, unless its data is abandoned. Its mariadb process will exit with an error. See [cluster behavior](cluster-behavior.html) for more details on intentionally abandoning data.

- On the new bootstrap node, update state file and restart the mariadb process:

  ```sh
  $ echo -n "NEEDS_BOOTSTRAP" > /var/vcap/store/mysql/state.txt
  $ monit start mariadb_ctrl
  ```

  You can check that the mariadb process has started successfully by running:

  ```sh
  $ watch monit summary
  ```

  It can take up to 10 minutes for monit to start the mariadb process.

- Once the bootstrapped node is running, start the mariadb process on the remaining nodes via monit.

  ```sh
  $ monit start mariadb_ctrl
  ```

- Verify that the new nodes have successfully joined the cluster. The following command should output the total number of nodes in the cluster:

  ```sh
  mysql> SHOW STATUS LIKE 'wsrep_cluster_size';
  ```
