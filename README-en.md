# **pg_monz - Template for monitoring PostgreSQL and PgPool**

This repository was a fork of the [pg_monz](https://github.com/pg-monz/pg_monz.git) project, which until now was the only project available to the community that really proposed monitoring the two tools together.
However, some time ago, about 2 years ago, the project was not updated and many issues ended up being forgotten.
Faced with my need for use, I was able to make changes to the codes in order to put the monitoring to work.
And this is the result of some efforts.

The version of zabbix used was 5.0 LTS, with PostgreSQL v15.3 and PgPool v4.4.

## Installation

1. Requirements:

- Zabbix server version 2.0+
- monitored PostgreSQL 9.2+
- monitored pgpool-II 3.4.0
- installing zabbix_agentd to PostgreSQL and pgpool-II host.
- installing zabbix_sender to PostgreSQL and pgpool-II host.
- ServerActive setting in zabbix_agentd.conf (zabbix_sender read this file)
- psql command(and serch path setting) on each PostgreSQL/pgpool server

2. Import monitoring templates into Zabbix Web.

   Template_App_PostgreSQL.xml
   Template_App_PostgreSQL_SR.xml
   Template_App_PostgreSQL_SR_Cluster.xml
   Template_App_pgpool-II.xml
   Template_App_pgpool-II-36.xml
   Template_App_pgpool-II_watchdog.xml

3. Copy the files from the `usr-local-etc` folder (pgsql_funcs.conf and pgpool_funcs.conf) to `/usr/local/etc`:

```bash 
  #cp usr-local-etc/* /usr/local/etc

```

Default values of variables from the `pgsql_funcs.conf` file

```bash
    ----
    PGHOST=127.0.0.1
    PGPORT=5432
    PGROLE=postgres
    PGDATABASE=postgres
    ----
```

Default values of variables from the `pgpool_funcs.conf` file
```bash
    ----
    PGPOOLHOST=127.0.0.1
    PGPOOLPORT=9999
    PGPOOLROLE=postgres
    PGPOOLDATABASE=postgres
    PGPOOLCONF=/usr/local/etc/pgpool.conf
    ----
```

> The definition of the configuration directory is done via the macro variable in the template.
```bash
       {$PGSCRIPT_CONFDIR}
       {$PGPOOLSCRIPTS_CONFDIR}
```

4. Copy the files from the `usr-local-bin` folder to the `/usr/local/bin`:

```bash
   #cp usr-local-bin/* /usr/local/bin
```

> The definition of the execution script directory is done via the macro variable in the template.
```bash
       {$PGSCRIPTDIR}
       {$PGPOOLSCRIPTDIR}
```

5. Copy the user parameter configuration file for the Zabbix agent `userparameter_pgsql.conf` to the specified location on the machine that has the agent installed.

For example, if the Zabbix agent is installed in `/etc/zabbix/`, copy the file to the following location:

```bash
cp zabbix_agentd.d/userparameter_pgpsql.conf /etc/zabbix/zabbix_agentd.conf.d/userparameter_pgsql.conf
```

> Make sure that `Include` for this folder is activated in the agent configuration file, as shown in the example below:
```bash
 Include=/etc/zabbix/zabbix_agentd.conf.d/
```

6. Defining groups using Zabbix Web.

- Create the "PostgreSQL" group and add the PostgreSQL Host to the created group.
- Create the "pgpool" group and add the pgpool-II host to the "pgpool" group.

> Each group is referenced in the Template_App_PostgreSQL_SR_Cluster.xml and Template_App_pgpool-II_watchdog.xml templates using the `{$HOST_GROUP}` variable in the template macro.

7. Check the directory path of the zabbix_agentd.conf file.

If the `zabbix_agentd.conf` file is not in `/etc/zabbix/zabbix_agentd.conf`, add the correct path as value in the `{$ZABBIX_AGENTD_CONF}` macro variable.

> Example of how to set `zabbix_agentd.conf` file path in macro variable in host or template.
```bash
     {$ZABBIX_AGENTD_CONF} => /etc/zabbix/zabbix_agentd.conf

```

8. Link templates to hosts.

Link "Template App PostgreSQL SR" for PostgreSQL hosts.

Link "Template App pgpool-II" for pgpool-II hosts.

"Template App PostgreSQL SR Cluster"/ "Template App pgpool-II-watchdog" are simply counting:

- the running service number (sr/pgpool-II),
- the main server number (sr),
- the standby server number (sr),
- the delegate_ip number (pgpool-II),

Using aggregate key in zabbix `{$HOST_GROUP}`.

> If you want to monitor split-brain or number of primary servers, just link above templates to arbitrary host.
> example: "PostgreSQL Cluster" as virtual host.

## Tips

Authentication in PostgreSQL and PgPool services can be done in 2 (two) ways, the first indicating the pgpass file inside the `pgsql_funcs.conf` file and the second way is by placing the pgpass to be used in the Zabbix user session.
I'll show both ways, I used pgpass on the Zabbix user and it worked just fine.

### First way:

Add the line below in the `pgsql_funcs.conf` file:
```bash
export PGPASSFILE=/usr/local/etc/pgpass
```

Create the `/usr/local/etc/pgpass` file with your favorite editor, and put the connection information in the pattern below:

```bash
127.0.0.1:5432:*:postgres:somepassword
```

After that give read-only permission on the file and also to the zabbix user.

```bash
chmod 0600 /usr/local/etc/pgpass
chown zabbix:zabbix /usr/local/etc/pgpass
```

### Second way:

Create the `/home/zabbix/.pgpass` file with the editor of your choice, and put the connection information in the pattern below:

```bash
127.0.0.1:5432:*:postgres:somepassword
```

After that give read-only permission on the file and also to the zabbix user.

```bash
chmod 0600 /home/zabbix/.pgpass
chown zabbix:zabbix /home/zabbix/.pgpass
```

Changing the permission is important because the file is created with the current user and thus the permissions are inherited from that user.

This form of configuration facilitates because it is already a practice used for connection via session using the psql utility.

pg_monz 2.0
============================
pg_monz (PostgreSQL monitoring template for Zabbix) is a Zabbix template for
monitoring PostgreSQL. It enables various types of monitoring of PostgreSQL
such as alive, resource, performance, etc.
Pg_monz also supports automatic discovery of databases and tables using the
discovery feature of Zabbix and can automatically start monitoring.


Changes from 1.0
----------------
The following is a summary of the major changes.


### Support for monitoring of PostgreSQL Streaming Replication
pg_monz 2.0 now supports monitoring of Streaming Replication which is embedded in PostgreSQL since 9.0.  
Various monitoring items such as Primary / Standby servers alive monitoring, delay of replication data propagation and conflicts occurred by operation to Primary and Standby are available.
It is also that a trigger which can detect the occurence of write block query when useing synchronous replication is provided.


### Support for monitoring of pgpool-II
pg_monz 2.0 now supports monitoring of pgpool-II which is a dedicated middleware for PostgreSQL.  
Various types of monitoring and triggers for the main features of pgpool-II such as Connection Pooling, Replication, In memory query Cache, Load Balance, Automatically Failover of PostgreSQL are provided.

Please see [pgpool-II user manual](http://www.pgpool.net/docs/latest/pgpool-en.html), [pgpool Wiki](http://www.pgpool.net/mediawiki/index.php/Main_Page) for more detailed informations.


### Support for monitoring of cluster system with PostgreSQL + pgpool-II
And more, it make it possible to monitor a cluster system which is configured with PostgreSQL Streaming replication and pgpool-II or pgpol-II watchdog which add high availability to themselves.  
Useful triggers which can detects the occurence of split brain, failover are provided through monitoring of postgres, pgpool-II processes.


### Group items
Monitoring items are grouped by each application to clarify them.  
The following are main applications.


#### Applications | PostgreSQL
|application name   |summary of monitoring                                                                            |
|:------------------|-------------------------------------------------------------------------------------------------|
|pg.transactions    |Connection count, state to PostgreSQL, the number of commited, rolled back transactions          |
|pg.log             |log monitoring for PostgreSQL                                                                    |
|pg.size            |garbage ratio, DB size                                                                           |
|pg.slow_query      |slow query count which exceeds the threshold value                                               |
|pg.sr.status       |conflict count, write block existence or non-existence, process count using Streaming Replication|
|pg.status          |PostgreSQL processes working state                                                               |
|pg.stat_replication|delay of replication data propagation using Streaming Replication                                |
|pg.cluster.status  |PostgreSQL processes count as a cluster                                                          |


#### Applications | pgpool-II
|application name   |summary of monitoring                                                                            |
|:------------------|-------------------------------------------------------------------------------------------------|
|pgpool.cache       |cash informations using In Memory query Cache                                                    |
|pgpool.connections |frontend, backend connection count through pgpool-II                                             |
|pgpool.log         |log monitoring for pgpool-II                                                                     |
|pgpool.nodes       |backend state, load balance ratio and replication delay viewed from pgpool-II                                          |
|pgpool.status      |pgpool-II processes working state, vip existence or non-existence                                |
|pgpool.watchdog    |pgpool-II processes working state, vip existence or non-existence as a cluster                   |


### Improve performance of gathering monitoring items
Previously, pg_monz accesses the monitoring DB every when gathering one monitoring item about DB, which may affect the performance of monitoring DB.
With this update, to reduce the frequency of DB accesse, pg_monz gathers collectable monitoring items all at once.


System requirements
-------------------
pg_monz requires the following software products:

* Zabbix server, zabbix agent, zabbix sender 2.0 or later
* PostgreSQL 9.2 or later
* pgpool-II 3.4.0 or later


Installation and usage
----------------------
Please see the included quick-install.txt.  
pg_monz 2.0 does not have backward compatibility with the 1.0. When upgrading from 1.0, please install the new version again.


License
-------
pg_monz is distributed under the Apache License Version 2.0.
See the LICENSE file for details.

Copyright (C) 2013-2021 SRA OSS, Inc. Japan All Rights Reserved.  
Copyright (C) 2013-2021 TIS Inc. All Rights Reserved.
