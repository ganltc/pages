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
any GPFS file system exports.  SELinux also prevents ganesha daemon
reading its own configuration files. See ganesha log for exact failure
case.

You need to disable SELinux or teach SELinux to allow ganesha.nfsd
daemon to open the /dev/ss0 character special file.

2. **Port already in use**

Historically, all NFS servers use port 2049 for NFS protocol (it is also
required for NFSv4). So you can't run two NFS servers at the same time.
You may have another instance of ganesha daemon already running or it is
also possible that Linux kernel NFS server is started with or without
your knowledge!

"systemctl mask nfs-server" would be better way to ensure that you don't
get linux kernel NFS server running!

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

   Run "systemctl status nfs-server" and "systemctl status nfs-ganesha"
   and see how and when they are started. "rpcinfo -p" gives the port
   numbers registered with portmapper. "lsof -i :<port>" or "ss -nlp |
   grep <port>" gives the current process on the system using the port.
   If you don't get correct process, then it could be owned by a kernel
   thread!

   The best way to guard against accidental start up of Linux kernel NFS server
   is to mask the service.  You can also disable it but then someone could
   accidentally start it, so we prefer to mask the Linux kernel NFS service
   (``systemctl mask nfs-server``).

NFS client or application hang due to NLM locks
================================================

Linux NFS client and kernel NFS server use the same network lock
manager.  nfs-ganesha has its own network lock manager. This means we
can't have Linux NFS client with an NFSv3 mount and nfs-ganesha running
at the same time. If you mount an NFS export with NFSv3 on the node
running ganesha, it would take NLM port preventing ganesha servicing any
NLM requests.

Ganesha registers only version 4 NLM for tcp and udp. So you only see two
nlockmgr lines in "rcpinfo -p" output::

 # rpcinfo -p | grep lockmgr
    100021    4   udp  50836  nlockmgr
    100021    4   tcp  33448  nlockmgr

The Linux kernel lock manager would register NLM for versions 1, 3 and
4.  A typical output will have 6 entries as below::

 # rpcinfo -p | grep lockmgr
    100021    1   udp  53568  nlockmgr
    100021    3   udp  53568  nlockmgr
    100021    4   udp  53568  nlockmgr
    100021    1   tcp  32770  nlockmgr
    100021    3   tcp  32770  nlockmgr
    100021    4   tcp  32770  nlockmgr

"tcpdump" trace would show all NLM requests getting rejects with the
following as kernel NLM client would only know how to make requests
(not reponses!)::

    Message Type: Reply (1)
    [Program: NLM (100021)]
    [Program Version: 4]
    [Procedure: LOCK (2)]
    Reply State: denied (1)
    [This is a reply to a request in frame 19]
    [Time from request: 0.000102000 seconds]
    Reject State: AUTH_ERROR (1)
    Auth State: bad credential (seal broken) (1)


Ganesha hangs 
=============

Ganesha coredumps
=================

tcpdump
=======
