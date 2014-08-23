---
title: Back Up MySQL for Pivotal CF
---

1. Locate the IP address for the MySQL node in the Status tab.

  ![MySQL Server IP](mysql-server-ip.png)

- Locate the root password for the MySQL server in the Credentials tab.

  ![MySQL Server Root Password](mysql-root-password.png)

-  Dump the data from the server and save it:

  <pre class="terminal">
  $ mysqldump -u root -p -h 172.16.78.60 --all-databases > user_databases.sql
  </pre>
