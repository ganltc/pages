==========================================================
Problem Determination Guide for Spectrum Scale NFS-Ganesha
==========================================================

:date: 2017-09-16
:modified: 2017-09-16
:tags: nfs
:category: nfs
:authors: Malahal Naineni
:summary: NFS-Ganesha problem determination.


Installation issues
===================

nfs-ganesha probject uses libntirpc as a subproject. We currently ship
libntirpc library code as part of the nfs-ganesha packages. Redhat ships
libntirpc and nfs-ganesha packages separately. If you already installed
libntirpc packages from EPEL repository, you will have same files conflicting
from IBM supplied nfs-ganesha packages. Please remove existing libntirpc
packages or de-activate yum/dnf repository supplying libntirpc package.

Other than the above issue, there shouldn't be any issues as long as you
installed on supported distributions!

Starting NFS-Ganesha service 
============================

**If you are unable to start nfs-ganesha service ganesha.log
(preferably with FULL_DEBUG mode) and syslog will provide us the most
valuable information.** Here are some common issues:

1. **SELinux**

SELinux in some Linux distros prevents nfs-ganesha daemon
(/usr/bin/ganesha.nfsd) to open /dev/ss0 which is required for exporting
any GPFS file system exports. You need to disable SELinux or teach
SELinux to allow ganesha.nfsd daemon to open the /dev/ss0 character
special file.

2. **Port already in use**

Historically, all NFS servers use port 2049 for NFS protocol (it is also
required for NFSv4). So you can't run two NFS servers at the same time.
You may have another instance of ganesha daemon already running or it is
also possible that Linux kernel NFS server is started with or without
your knowledge!

If you want to know the current process using port 2049, you could do::

        lsof -i :2049


Issues mounting exports from NFS clients
========================================

So you have ganesha NFS server running, but unable to mount an export
that you think you should. **Network trace is your best friend here. Actually
network trace is your best friend for any problem post start up!** Here
goes a list of issue you may encounter:

1. **NFSv3 mount failure due to portmapper**

   Portmapper (mapping of services to ports) service runs at port 111.
   Your client will send GETPORT request for mountd service to get the
   mountd port number on the NFS server node. NFS mount will fail if
   your server doesn't return mountd port for one reason or the other. A
   classic case is that you start Linux kernel NFS server after starting
   nfs-ganesha server. The Linux kernel NFS server will register it own
   mountd port overwriting nfs-ganesha mountd port.  Eventually, the
   Linux kernel NFS server will fail to come up as it can't bind to 2049
   NFS port, but the damage has already been done.

   Run "systemctl status nfs" and "systemctl status nfs-ganesha" and see
   how and when they are started. "rpcinfo -p" gives the port numbers
   registered with portmapper. "lsof -i :<port>" gives the current
   process on the system using the port.

   The best way to guard against accidental start up of Linux kernel NFS server
   is to unmask the service.  You can also disable it but then someone could
   accidentally start it, so we prefer to mask the Linux kernel NFS service
   (``systemctl mask nfs`` on Redhat based distros and ``systemctl mask
   nfs-kernel-server`` on debian based distros) service!

2. **NFSv3 locking failure**

NFS client or application hangs
===============================

Linux NFS client and kernel NFS server use the same network lock manager.
nfs-ganesha has its own network lock manager. This means we can't have Linux
NFS client with an NFSv3 mount and nfs-ganesha running at the same time. If you mount an NFS
export 



Ganesha hangs 
=============

Ganesha coredumps
=================

tcpdump
=======
