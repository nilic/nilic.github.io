---
layout: post
comments: true
title: Trying out Docker client for Windows
---

Docker client for Windows was [released a few months ago](https://azure.microsoft.com/blog/2015/04/16/docker-client-for-windows-is-now-available/), and recently I installed it inside of a Windows 10 CTP machine and tested it against a CentOS 7 serving as a Docker host.

For the installation I employed the [Chocolatey](https://chocolatey.org/) package manager:

```
C:\> choco install docker -y
```

which will install the latest version of the client (1.7.0 at the time of writing).

By default, docker daemon listens only on `unix:///var/run/docker.sock` socket and therefore accepts only local connections, so in order to access it from the outside I needed to bind it to a TCP port. So I first stopped the Docker daemon on my container host:

```
$ sudo systemctl stop docker
```

and then ran it like this so it binds to TCP port 2375 in addition to the default socket:

```
$ sudo docker -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -d &
```

**Note:** This was only for the purpose of a quick-and-dirty test, and you can read [here](https://docs.docker.com/articles/basics/) why allowing the daemon to accept remote calls in this way isn't such a great idea security-wise.

After this, everything was ready for connecting from my Windows client, or so I thought. I tried listing images on the docker host but was greeted with the following error:

```
C:\> docker -H 10.0.0.1:2375 info
Error response from daemon: client and server don't have same version (client : 1.19, server: 1.18)
```

After a quick `docker -v`, I realized that the daemon was running Docker version 1.6.2, while the client was running version 1.7.0.

I had no choice but to fire up Chocolatey again and install version 1.6.0 of the client:

```
C:\> choco uninstall docker -y
C:\> choco install docker -version 1.6.0 -y
```

and then everything worked as expected:

```
C:\> docker -H 10.0.0.1:2375 info
Containers: 0
Images: 33
Storage Driver: devicemapper
...
Kernel Version: 3.10.0-123.el7.x86_64
Operating System: CentOS Linux 7 (Core)
CPUs: 1
Total Memory: 458.4 MiB
Name: dockerhost
ID: DBST:ZQ5O:LOAJ:XG7D:FEBL:6FYO:2JZ3:CQN2:QPK6:6ARN:XBEZ:THVQ
```

If you go back to the Docker host, you'll notice that your client commands result in calls to the Docker API:

```
...
INFO[0680] GET /v1.18/info                              
...
```

where `v1.18` is the API version and when a client is running on a higher version than the daemon, a mismatch occurs in the API calls (trying to access `/v1.19/info` instead of `/v1.18/info` and similar). This explains the problem I originally had, although at the time it seemed kinda odd to me that you cannot use a higher version client to access a lower version daemon. 

In order to avoid having to specify your Docker host parameters with every command, you can set a Windows environment variable `DOCKER_HOST` and assign it a value of `tcp://<FQDN or IP of Docker host>:<TCP port>`, e.g. `tcp://10.0.0.1:2375` in my case. Afterwards, you can run the commands on your Windows client machine as if you're located directly on the Docker host:

```
C:\> docker run -it ubuntu
root@825574d22c14:/# cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=14.04
DISTRIB_CODENAME=trusty
DISTRIB_DESCRIPTION="Ubuntu 14.04.1 LTS"
root@825574d22c14:/# exit
C:\>
```

Pretty crazy, huh :) Kudos to the Microsoft Azure Linux team for porting the Docker client to Windows, and be sure to read more about their efforts on [Ahmet Alp Balkan's blog](https://ahmetalpbalkan.com/blog/porting-docker-client-to-windows/).
