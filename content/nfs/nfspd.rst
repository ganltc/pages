==========================================================
Problem Determination Guide for Spectrum Scale NFS-Ganesha
==========================================================

:date: 2017-09-22
:modified: 2018-07
:tags: nfs
:category: nfs
:authors: Malahal Naineni
:summary: NFS-Ganesha problem determination.


Installation issues
===================

NFS-Ganesha project uses libntirpc as a subproject. We currently ship
libntirpc library code as part of the NFS-Ganesha packages. Redhat ships
libntirpc and NFS-Ganesha packages separately. If you already installed
libntirpc packages from EPEL repository, you will have same files conflicting
from IBM supplied NFS-Ganesha packages. Please remove existing libntirpc
packages or de-activate yum/dnf repository supplying libntirpc package.

Other than the above issue, there shouldn't be any issues as long as you
installed on supported distributions!

Starting NFS-Ganesha service 
============================

**If you are unable to start nfs-ganesha service ganesha.log
(preferably with FULL_DEBUG mode) and syslog will provide us the most
valuable information.** Here are some common issues:

1. **SELinux**

SELinux in some Linux distros prevents NFS-Ganesha daemon
(/usr/bin/ganesha.nfsd) to open /dev/ss0 which is required for exporting
any GPFS file system exports.  SELinux also prevents NFS-Ganesha daemon
reading its own configuration files! See NFS-Ganesha log for exact failure
case.

You need to disable SELinux or teach SELinux to allow ganesha.nfsd
daemon to open the /dev/ss0 character special file.

2. **Port already in use**

Historically, all NFS servers use port 2049 for NFS protocol (it is also
required for NFSv4). So you can't run two NFS servers at the same time.
You may have another instance of NFS-Ganesha daemon already running or it is
also possible that Linux kernel NFS server is started with or without
your knowledge!

"systemctl mask nfs-server" would be better way to ensure that you don't
get linux kernel NFS server running!

Issues mounting exports from NFS clients
========================================

If you have NFS-Ganesha server running, but unable to mount an export
that you think you should. **Network trace is your best friend here. Actually
network trace is your best friend for any problem post start up!** Here
goes a list of issue you may encounter:

1. **NFSv3 mount failure due to portmapper**

   Portmapper (mapping of services to ports) service runs at port 111.
   Your client will send GETPORT request for mountd service to get the
   mountd port number on the NFS server node. NFS mount will fail if
   your server doesn't return mountd port for one reason or the other. A
   classic case is that you start Linux kernel NFS server after starting
   NFS-Ganesha server. The Linux kernel NFS server will register it own
   mountd port overwriting NFS-Ganesha mountd port.  Eventually, the
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


2. ** security settings **

   NFV4 is affected more than NFSv3 due to NFSv4 pseudo traversal. If /A
   is exported with 'krb5' only authentication and subdirectory /A/B is
   exported with 'SYS' only authentication, mount of '/A/B' would fail
   unless the NFS client tries krb5 for /A and then 'SYS' for /A/B.


NFS client or application hang due to NLM locks
================================================

Linux NFS client and kernel NFS server use the same network lock
manager.  NFS-Ganesha has its own network lock manager. This means we
can't have Linux NFS client with an NFSv3 mount and NFS-Ganesha running
at the same time. If you mount an NFS export with NFSv3 on the node
running NFS-Ganesha it would take NLM port preventing NFS-Ganesha
servicing any NLM requests.

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


Throttling log messages
=======================

Some NFS-Ganesha threads read socket data, prepare protocol requests
with the help of other threads, and then place the requests in request
queues. NFS-Ganesha worker threads dequeue such requests from the
request queues for processing. If NFS-Ganesha worker threads are slow in
processing, the request queues get larger and larger. Instead of
allowing more and more requests from the NFS clients, the socket reading
threads throttle reading socket data there by throttling NFS clients
sending NFS requests.

NFS-Ganesha logs the following messages when request queue lengths hit
the configured levels. The first message indicates the global counter
(total requests from all sockets) reaching its limit, so all sockets are
throttled. The second message indicates a given transport reaching its
configured limit and throttles only the corresponding transport::
 
    2016-11-01 19:48:01 : epoch 000b0042 : localhost : ganesha.nfsd-16645[disp] nfs_rpc_getreq_ng :DISP :EVENT :global outstanding reqs quota exceeded (have 5008, allowed 5000)

    2017-12-21 01:55:41 : epoch 00060217 : localhost : ganesha.nfsd-12600[disp] nfs_rpc_cond_stall_xprt :DISP :EVENT :xprt 0x7f99b803a500 has 5001 reqs, marking stalled

Both the above messages usually indicate a slow back end (aka any of
GPFS, Network, or Storage).  Other reason could be a hung NFS-Ganesha
worker threads. If periodic execution of "ganesha_stats" show increased
number of processed operations, then it is unlikely to be an NFS-Ganesha
hang.

Ganesha hangs 
=============
- tcpdump should be collected to determine a possible root cause
- tcpdump from both sides (NFS client and NFS server) would be good
- Always use pcap format while capturing the data (use -w <filename>
  option with tcpdump command)
- Full packet capture should be done for hangs (-s0)
- Use ganesha_mgr to capture NFS-Ganesha traces
- Enable GPFS tracing (vnode level 5)
- A forced coredump of NFS-Ganesha daemon

See `data collection for hang analysis
<{filename}./data.collect.perf.hang.rst>`_ for more details..
