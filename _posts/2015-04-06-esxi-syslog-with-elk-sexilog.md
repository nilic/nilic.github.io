---
layout: post
comments: true
title: Centralized ESXi syslog with ELK feat. SexiLog
---

The ELK stack is a commonly used open source solution for log collection and analysis. It consists of three components:

* Elasticsearch - distributed search engine, responsible for storing and searching through the received data;

* Logstash - log collector, parser and forwarder;

* Kibana - Elasticsearch web frontend, the end user interface for searching and visualizing log data.

There are a lot of tutorials on setting up the ELK stack, but if you're looking into implementing ELK as a centralized (sys)log server for your vSphere environment, you should probably look no further than [SexiLog](http://www.sexilog.fr/).

To quote the official site - _SexiLog is a specific ELK virtual appliance designed for vSphere environment. It’s pre-configured and heavily tuned for VMware ESXi logs._ This means that the nice folks from [Hypervisor.fr](http://www.hypervisor.fr/) and [VMdude.fr](http://www.vmdude.fr/) have gone through the trouble of installing ELK, optimizing it for vSphere log collection, adding a bunch of other useful tools and packaging everything as an easy to deploy VMware virtual appliance.

The appliance is a Debian-based VM with 2vCPU, 4GB RAM and 58GB hard drive space (8GB system disk + 50GB disk for storing indexed logs), and is sized for collecting up to 1500 events per second. Default credentials for the appliance are `root` / `Sex!Log` and after each log in you will be greeted with a menu that can be used for basic configuration and operations. Since this "SexiMenu" is a bash script which can be run manually from `/root/seximenu/seximenu.sh`, if you want to avoid it and log in directly to shell each time, you can simply comment out the last three lines from `/root/.bashrc`.

After deploying the appliance, you'll need to configure your ESXi hosts to start sending syslog data, which can be done manually through [vSphere client or ESXCLI](http://kb.vmware.com/kb/2003322) or in an automated manner, e.g. with [PowerCLI](http://blogs.vmware.com/vsphere/2013/07/log-insight-bulk-esxi-host-configuration-with-powercli.html). Besides syslog, SexiLog also offers the possibility of collecting SNMP traps (and has Logstash SNMP grok patterns optimized for ESXi and Veeam Backup and Replication), as well as vCenter `vpxd` logs and Windows event logs with the help of the [NXLog agent](http://nxlog.co/products/nxlog-community-edition). For more info, [RTFM](http://www.sexilog.fr/rtfm/) :)

## SexiLog goodies

SexiLog offers much more than just a vanilla ELK installation. Besides providing Logstash filters optimized for vCenter, ESXi and Veeam B&R log and SNMP trap collection, it also comes with a bunch of [pre-configured Kibana dashboards](http://www.sexilog.fr/sexiboards/).

Then, there is a notification service powered by [Riemann](http://riemann.io/), which receives warnings and alerts from Logstash and either sends them every 10 minutes (critical alerts) or aggregates them and e-mails them once per hour (all other alerts). You can check out which events are considered critical by looking at Logstash configuration files located at `/etc/logstash/conf.d/` and searching for rules which add the _achtung_ tag. In order for Riemann to do its job, it is necessary to configure SMTP parameters for the appliance, which can be done through the SexiMenu (option 7).

Another important part of SexiLog is [Curator](https://github.com/elasticsearch/curator), which is used for purging Elasticsearch indices, in order for them not to fill up the `/dev/sdb1` partition used for storing logs. Curator runs once per hour, as defined in `/etc/crontab`:

```
5 * * * * root curator delete --disk-space 40
```

and takes maximum allowed size of indices in GB as the input parameter. By default this is set to 40[GB] and should be changed if you decide to extend SexiLog's hard disk no. 2.

Also, for more info about the health and performance of the Elasticsearch service,  SexiLog provides two useful plugins - [Head](https://github.com/mobz/elasticsearch-head) and [Bigdesk](http://bigdesk.org/), which can be accessed from `http://<sexilog_IP_or_FQDN>/_plugin/head` and  `http://<sexilog_IP_or_FQDN>/_plugin/bigdesk`.

## Feature requests

Two possible improvements have come to my mind while using SexiLog. First, it would be nice if there was a way to provide user authentication for accessing the Kibana interface (maybe [kibana-authentication-proxy](https://github.com/fangli/kibana-authentication-proxy) could help with this since [Shield](https://www.elastic.co/products/shield) seems to be a commercial product?). Also, an option to export search results would be useful (e.g. when dealing with techical support), but this seems to be a current limitation of Elasticsearch/Kibana.

## Further steps

For more information about SexiLog check out the [site](http://sexilog.fr/), and be sure to try out the [demo](http://demo.sexilog.fr/). You can also contribute to this great project via [Github](https://github.com/sexilog/sexilog).