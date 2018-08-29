=====================================
Performance monitoring of NFS Ganesha
=====================================

:date: 2018-08-29
:modified: 2018-08-29
:tags: nfs, performance
:category: nfs
:authors: Sachin Punadikar
:summary: This article helps understand “ganesha_stats” utility and how to use it for performance monitoring.


NFS server implementation is available in two kinds, kernel space (the default
NFS server, available with almost all OS) and user space (NFS Ganesha). NFS
Ganesha is open source community project, and available at
https://github.com/nfs-ganesha/nfs-ganesha

IBM Spectrum Scale support NFS Ganesha as the default NFS server on protocol nodes (or CES nodes). “ganesha_stats” utility helps in understanding NFS server statistics. This utility is not cluster aware and provides information on NFS server running on that node only.

Usage of “ganesha_stats” can be divided into 2 broad categories. 

- First category is about extracting various statistics under different heads like per protocol version, per export, fsal specific etc. In general these statistics are again grouped into 3 categories, server specific, FSAL specific and RPC Queue specific. For FSAL specific stats use option “fsal” and for RPC Queue specific statistics use 
- The second category is about managing the stats counting operations such as enabling/disabling/resetting etc.


To get short information on the usage of “ganesha_stats”, pass “help” argument
as shown below::

        [root@c72n3 ~]# ganesha_stats help
        Command displays global stats by default.
        To display current status regarding stat counting use
        /usr/bin/ganesha_stats status
        To display stat counters use
        /usr/bin/ganesha_stats [list_clients | deleg <ip address> | inode | iov3 [export id] | iov4 [export id] | export | total [export id] | fast | pnfs [export id] | fsal <fsal name> ] | rpc
        To reset stat counters use
        /usr/bin/ganesha_stats reset
        To enable/disable stat counters use
        /usr/bin/ganesha_stats [enable | disable] [all | nfs | fsal | rpc]
        To get the current memory pool allocation
        /usr/bin/ganesha_stats pool


Options to extract stats
------------------------

- global – this is the default option, if no argument is passed. This option provides a global view of NFS server operation statistics related to different protocol versions.
- list_clients – this option provides information about all connected clients and also mentions whether per protocol are there any stats available or not.
- deleg – this option provides information on delegations related to the specified client. This option needs additional argument specifying the client IP address. Delegations are not supported by NFS Ganesha on Spectrum Scale.
- inode – this option provides information on caching done by NFS Ganesha. The information includes cache hit, miss, conflict etc.
- iov3 – This option provides information on NFSv3 related stats. If NFSv3 stats are available for an export, it provides details such as number requests, number of errors, latency etc.
- iov4 - This option provides information on NFSv4 related stats. If NFSv3 stats are available for an export, it provides details such as number requests, number of errors, latency etc.
- export – this option provide information on the exports defined. Export id, export path, whether the export has any stats related to various protocols like NFSv3, NFSv4, NLM, 9P etc.
- total – this option provides information on all exports and provides number of operations done per protocol version (NFSV3, NFSv40, NFSv41 etc)
- fast – this option provides operations count per type of operation per protocol for the server.
- pnfs – this option provides NFSv4.1 specific statistics. NFSv4.1 is not supported by NFS Ganesha on Spectrum Scale.
- fsal – this option provides information about FSAL specific stats counting. This option requires additional parameter, which is FSAL name (e.g. gpfs). FSAL specific stats counting is disabled by default.
- rpc – this option provides information about RPC queue related stats. RPC Queue specific stats counting is disabled by default.

Options to manage stats counting
--------------------------------

- help – this argument provides short help in “Ganesha_stats” utility.
- status – this option provides information about what all stats counting is
  enabled and since when. The different types of stats counting possible are
  nfs server, FSAL specific (fsal) and RPC Queue (rpc)
- reset – this option helps in resetting all the stats counters (NFS serevr,
  FSAL specific & RPC Queue specific) to 0. This is helpful in case one wants
  to check the performance for a specific run.
- enable/disable – these 2 options helps in dynamically enabling or disabling
  stats counting. Both options requires additional argument specifying which
  stats counting needs to be enabled/disabled. The valid options are NFS server
  (nfs), FSAL specific (fsal), RPC Queue Specific (rpc) or all.
- pool – this option is for getting information on various memory pool
  allocations. This is useful in understanding the current memory allocation in
  Ganesha (only via memory pools).


Steps to monitor the performance:
---------------------------------
Below procedure is required to be executed on all protocol (CES) node

1. To begin with, first enable stats counting for FSAL, RPC Queue (it is disabled by default):
    # ganesha_stats enable all
2. Before going to start the tests, make all the counters zero.
    # ganesha_stats reset
3. Start the tests. And periodically collect below stats on every node. (say for every 30 min run all commands & collect the o/p in a file)
    a) # ganesha_stats iov4
    b) # ganesha_stats iov3
    c) # ganesha_stats fsal gpfs
    d) # ganesha_stats rpc
    e) # ganesha_stats reset  <= Make all counters 0


Few Examples:
=============

**Checking current status of various stats counting:**
::

        [root@c72n3 ~]# ganesha_stats  status
        Stats counting for NFS server is enabled since:
                Thu Aug 23 00:40:19 2018658522215 nsecs
        Stats counting for FSAL is currently disabled
        Stats counting for RPC is currently disabled


**Enabling fsal specific stats:**
::

        [root@c72n3 ~]# ganesha_stats  enable fsal
        Successfully enabled statistics counting


**Enabling RPC Queue specific stats:**
::

        [root@c72n3 ~]# ganesha_stats  enable rpc
        Successfully enabled statistics counting

        [root@c72n3 ~]# ganesha_stats  status
        Stats counting for NFS server is enabled since:
                Thu Aug 23 00:40:19 2018658522215 nsecs
        Stats counting for FSAL is enabled since:
                Thu Aug 23 00:43:53 2018326619720 nsecs
        Stats counting for RPC is enabled since:
                Thu Aug 23 00:44:12 201879056004 nsecs

*Additional examples:*
::

        [root@c72n3 ~]# ganesha_stats  enable all
        Successfully enabled statistics counting

        [root@c72n3 ~]# ganesha_stats status
        Stats counting for NFS server is enabled since:
                Thu Aug 23 01:04:44 2018410148736 nsecs
        Stats counting for FSAL is enabled since:
                Thu Aug 23 01:04:44 2018410159670 nsecs
        Stats counting for RPC is enabled since:
                Thu Aug 23 01:04:44 2018410167175 nsecs



**Disabling stats (various examples):**
::

        [root@c72n3 ~]# ganesha_stats  disable fsal
        Successfully disabled statistics counting

        [root@c72n3 ~]# ganesha_stats  disable rpc
        Successfully disabled statistics counting

        [root@c72n3 ~]# ganesha_stats status
        Stats counting for NFS server is enabled since:
                Thu Aug 23 00:40:19 2018658522215 nsecs
        Stats counting for FSAL is currently disabled
        Stats counting for RPC is currently disabled

        [root@c72n3 ~]# ganesha_stats  disable nfs
        Successfully disabled statistics counting

        [root@c72n3 ~]# ganesha_stats status
        Stats counting for NFS server is currently disabled
        Stats counting for FSAL is currently disabled
        Stats counting for RPC is currently disabled


**Extracting protocol version specific stats:**
::

        [root@c72n3 ~]# ganesha_stats iov3 10

        EXPORT 10:
                requested       transferred          total          errors        latency      queue wait
        READv3:         189431808       189273602           2958               0        396821029342    474614894
        WRITEv3:        6021017356      6021017356         18905               0        429731879788    2600329223

        [root@c72n3 ~]# ganesha_stats iov4 12
        EXPORT 12:
                requested       transferred          total          errors         latency      queue wait
        READv4:         66838528        66443521             544               0        66705975273     81595943
        WRITEv4:        11078495378     11078495378        91229               0        1612120083701   11350539041



**Extracting fsal specific stats (considering fsal stats counting already enabled):**
::

        [root@c72n3 ~]# ganesha_stats fsal gpfs
        Timestamp: Thu Aug 23 00:37:14 2018554766976 nsecs
        FSAL Name - GPFS
        FSAL Stats (response time in milliseconds):
                Op-Name         Total     Res:Avg         Min           Max
        NAME_TO_HANDLE             45     0.037271     0.001532     1.460589
        OPEN_BY_HANDLE             57     1.226331     0.005179    59.483080
        GET_XSTAT                 436     0.027083     0.001211     9.457173
        SET_XSTAT                  12     5.177517     0.836911    20.598806
        CLOSE_FILE                 10     0.014505     0.006658     0.064869
        RENAME_BY_FH                1    85.685211    85.685211    85.685211
        STAT_BY_NAME                6     0.007922     0.006331     0.011113
        UNLINK_BY_NAME              5     5.621457     1.163010    12.988609
        CREATE_BY_NAME              5    30.377444     1.020835   101.153596
        READ_BY_FD                  2     0.015599     0.012296     0.018902
        WRITE_BY_FD                 3    11.512359     1.477251    28.555763



**Extracting Getting RPC Queue specific stats (assuming it is already enabled):**
::

        [root@c72n3 ~]# ganesha_stats  rpc
        RPC Queue Statistics (4 Queues):
         Total Requests :     27717
         Active Requests:         0
         QueueName              Total   Active   Min Res Time Max Res Time (Milliseconds)
        REQ_Q_MOUNT                 0        0      0.000000      0.000000
        REQ_Q_CALL                  0        0      0.000000      0.000000
        REQ_Q_LOW_LATENCY       27621        0      0.006925      1.122947
        REQ_Q_HIGH_LATENCY         96        0      0.007489      0.665582


**Resetting the stats:**
::

        [root@c72n3 ~]# ganesha_stats  reset
        Successfully resetted statistics counters


