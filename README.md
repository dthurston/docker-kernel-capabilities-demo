# Docker Kernel Capabilities Demo
Container inheriting host security -- This is a simple two part demo that shows how a container inherits the underlying controls from the host kernel 

**Pre-requisites**

Boot up STIG'ed RHEL 7 baseline

Verify that docker is installed and running

```
subscription-manager repos --enable=rhel-7-server-extras-rpms
subscription-manager repos --enable=rhel-7-server-optional-rpms
yum install docker device-mapper-libs device-mapper-event-libs
systemctl start docker.service
systemctl enable docker.service
```
Verify the docker service is running

```
systemctl status docker.service
```

**Docker read only Filesystem in /proc/sys**

Download a sample container to work on (it really doesn’t matter which one afaik)

Purpose:  To show that containers /proc/sys filesystem are read only by default on RHEL7

```
docker pull registry.access.redhat.com/rhscl/python-34-rhel7
docker run -it rhscl/python-34-rhel7 /bin/bash
[in container] cat /proc/sys/kernel/modules_disabled
[in container] echo 1 > /proc/sys/kernel/modules_disabled
```

You will get an error showing that the container is a read only file system.  No one could use /proc/sys to manipulate the container for evil purposes.

```
exit
```

Open a root terminal for docker and for host
```
docker run -it --user root rhscl/python-34-rhel7 /bin/bash
[in container] echo 1 > /proc/sys/kernel/modules_disabled
[outside container] cat /proc/sys/kernel/modules_disabled
```
When you launch a container as root, you still cannot change /proc/sys

**Launching a container with reduced capabilities**

Purpose:  To show that kernel capabilities can be assigned to Docker containers

Launch a normal container
```
docker run -it rhscl/python-34-rhel7 /bin/bash
ping 127.0.0.1
```
You are able to ping 127.0.0.1.  

Now reduce the capabilities of the container.
```
docker run -it --cap-drop net_admin --cap-drop net_raw rhscl/python-34-rhel7 /bin/bash
ping 127.0.0.1
```
Ping will now fail.
```
exit
```
References:

Docker required capabilities: https://github.com/docker/docker/blob/master/oci/defaults_linux.go#L64-L79

Linux Kernel capabilities: http://man7.org/linux/man-pages/man7/capabilities.7.html
