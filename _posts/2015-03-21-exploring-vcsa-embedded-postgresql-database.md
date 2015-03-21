---
layout: post
comments: true
title: Exploring VCSA embedded PostgreSQL database
---

Since vSphere 5.0U1, VMware vCenter Server Appliance (VCSA) uses vPostgres - VMware flavored PostgreSQL as the embedded database. This post describes how to connect to the VCSA vPostgres server locally and remotely, and perform database backups using native PostgreSQL tools.

__Note:__ Following procedures are probably unsupported by VMware and are given here just for the fun of hacking around VCSA. The instructions have been tested for VCSA 5.5 and 6.0 (VCSA 6.0 requires additional steps which can be found at the bottom of the post).

## Connecting to PostgreSQL server locally

After logging to the VCSA over SSH or console, you can easily connect to the PostgreSQL server locally using `psql`:

`/opt/vmware/vpostgres/current/bin/psql -U postgres`

After connecting you can use psql or regular SQL commands, e.g.

```
# /opt/vmware/vpostgres/current/bin/psql -U postgres
psql.bin (9.3.5 (VMware Postgres 9.3.5.2-2444648 release))
Type "help" for help.

postgres=# \du
                             List of roles
 Role name |                   Attributes                   | Member of 
-----------+------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication | {}
 vc        | Create DB                                      | {}
```

Here we see that there are two users defined in the PostgreSQL server, the `postgres` superuser, and the `vc` user, used by vCenter for connecting to its database.

## Enabling remote PostgreSQL server access

By default, only local connections to the database server are allowed. In order to allow remote access (e.g. so that you can connect via GUI based administrative tools such as [pgAdmin](http://www.pgadmin.org/)), first take a look at the following files:

`/etc/vmware-vpx/embedded_db.cfg`

`/etc/vmware-vpx/vcdb.properties`

`embedded_db.cfg` file stores general PostgreSQL server information (as well as the password for the `postgres` superuser), while `vcdb.properties` stores connection information for the vCenter server database `VCDB`, along with the password for the `vc` user. Take a note of these passwords, since you'll be required to supply them for remote access.

Then, edit the `/storage/db/vpostgres/pg_hba.conf` configuration file in order to allow your IP to connect to the PostgreSQL server by adding the following line:

```
host    all             all             1.2.3.4/24          md5
```

replacing `1.2.3.4/24` with the actual IP address or range of addresses for which you want to allow access (e.g. `192.168.1.0/24`).

Next, edit the `/storage/db/vpostgres/postgresql.conf` in order to configure PostgreSQL server to listen on all available IP addresses by adding the following line:

```
listen_addresses = '*'
```

Finally, restart the PostgreSQL server by running

`/etc/init.d/vmware-vpostgres restart`

## Backing up the vCenter database

[VMware KB 2034505](kb.vmware.com/kb/2034505) provides information on using native PostgreSQL tools to perform VCDB backups and restores. The requirement for the vCenter service to be stopped during the database backup seems kinda redundant, since `pg_dump` should perform consistent backups even if the database is in use.

Sample backup scripts and instructions on how to schedule them via `cron` can be found on [Florian Bidabe's](http://bidabe.zapto.org/?p=360) and [vNinja](http://vninja.net/virtualization/vpostgres-database-backup-vcsa-5-5/) blogs. Since `mount.nfs` is available on the VCSA, it seems that you can even use an NFS share as a destination for your VCDB backups (haven't tested it though).

## VCSA 6.0 additional steps

VCSA 6.0 comes extra hardened compared to previous vSphere editions and additional steps are needed in order to allow remote access to the OS and then to the PostgreSQL server.

First, you need to enable SSH access to VCSA. This can be done during the deployment, or later, over VM console (similar interface to ESXi DCUI: __F2__ - __Troubleshooting Mode Options__ - __Enable SSH__) or vSphere Web Client (__Home__ - __System Configuration__ - __Nodes__ - __Manage__ - __Settings__ - __Access__).

After logging as `root` over SSH, you will be greeted with a limited shell called `appliancesh`. To switch to `bash` temporarily, run:

```
shell.set --enabled True
shell
```

For switching to `bash`  permanently you can follow the instructions from [this virtuallyGhetto post](http://www.virtuallyghetto.com/2015/03/how-to-changedeploy-vcsa-6-0-with-default-bash-shell-vs-appliancesh.html).

The final step is to allow external access to the PostgreSQL through the VCSA IPTables-based firewall. This can be done by editing the `/etc/vmware/appliance/firewall/vmware-vpostgres` file so that it looks like this:

```
{
  "firewall": {
    "enable": true,
    "rules": [
      {
        "direction": "inbound",
        "protocol": "tcp",
        "porttype": "dst",
        "port": "5432",
        "portoffset": 0
      }
    ]
  },
  "internal-ports": {
    "rules": [
      {
        "name": "server_port",
        "port": 5432
      }
    ]
  }
}
```

Afterwards, reload VCSA firewall by running

`/usr/lib/applmgmt/networking/bin/firewall-reload`

and PostgreSQL server should be accessible from the outside world after configuring `pg_hba.conf` and `postgresql.conf` as described above.