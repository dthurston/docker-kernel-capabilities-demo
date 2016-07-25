# Docker Kernel Capabilities Demo
### Container inheriting host security -- This is a simple demo that shows how a container inherits the underlying controls from the host kernel 
**Demo Steps**
**Pre-requisites**
### Boot up STIG'ed RHEL 7 baseline
### Verify that docker is installed and running
'''bash
subscription-manager repos --enable=rhel-7-server-extras-rpms
subscription-manager repos --enable=rhel-7-server-optional-rpms
yum install docker device-mapper-libs device-mapper-event-libs
systemctl start docker.service
systemctl enable docker.service
'''
### Verify the docker service is running
'''bash
systemctl status docker.service
'''

Docker Read only FS in /proc/sys
# Download a sample container to work on (it really doesn’t matter which one afaik)
# Purpose:  To show that containers /proc/sys filesystem are read only by default on RHEL7
docker pull registry.access.redhat.com/rhscl/python-34-rhel7
docker run -it rhscl/python-34-rhel7 /bin/bash
[in container] cat /proc/sys/kernel/modules_disabled
[in container] echo 1 > /proc/sys/kernel/modules_disabled
# Here you get an error showing that the container is a read only file system.  No one could use /proc/sys to manipulate the container for evil purposes.
exit
# open a root terminal for docker and for host
docker run -it --user root rhscl/python-34-rhel7 /bin/bash
[in container] echo 1 > /proc/sys/kernel/modules_disabled
[outside container] cat /proc/sys/kernel/modules_disabled
# When you launch a container as root, you still cannot change /proc/sys

Launching a container with reduced capabilities
# First launch a normal container
docker run -it rhscl/python-34-rhel7 /bin/bash
ping 127.0.0.1
# You are able to ping 127.0.0.1.  
# Now reduce the capabilities of the container.
docker run -it --cap-drop net_admin --cap-drop net_raw rhscl/python-34-rhel7 /bin/bash
ping 127.0.0.1
# ping should fail here now.
exit

Kernel capabilities that docker can manage….
[root@docker ~]# docker run -it --cap-drop 
ALL               IPC_LOCK          NET_BROADCAST     SYS_MODULE
AUDIT_CONTROL     IPC_OWNER         NET_RAW           SYS_NICE
AUDIT_READ        KILL              SETFCAP           SYS_PACCT
AUDIT_WRITE       LEASE             SETGID            SYS_PTRACE
BLOCK_SUSPEND     LINUX_IMMUTABLE   SETPCAP           SYS_RAWIO
CHOWN             MAC_ADMIN         SETUID            SYS_RESOURCE
DAC_OVERRIDE      MAC_OVERRIDE      SYS_ADMIN         SYS_TIME
DAC_READ_SEARCH   MKNOD             SYS_BOOT          SYS_TTY_CONFIG
FOWNER            NET_ADMIN         SYS_CHROOT        WAKE_ALARM
FSETID            NET_BIND_SERVICE  SYSLOG            


