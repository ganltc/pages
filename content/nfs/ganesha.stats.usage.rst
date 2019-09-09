=====================================
Performance monitoring of NFS Ganesha
=====================================

:date: 2018-08-29
:modified: 2019-01-22
:tags: nfs, performance
:category: nfs
:authors: Sachin Punadikar
:summary: This article helps understand “ganesha_stats” utility and how to
    use it for performance monitoring.


NFS server implementation is available in two kinds, kernel space (the default
NFS server, available with almost all OS) and user space (NFS Ganesha). NFS
Ganesha is open source community project, and available at
https://github.com/nfs-ganesha/nfs-ganesha

IBM Spectrum Scale support NFS Ganesha as the default NFS server on protocol
nodes (or CES nodes). “ganesha_stats” utility helps in understanding NFS
server statistics. This utility is not cluster aware and provides information
on NFS server running on that node only.

Usage of “ganesha_stats” can be divided into 2 broad categories. 

- First category is about extracting various statistics under different heads
  like per protocol version, per export, fsal specific etc. In general these
  statistics are again grouped into 3 categories, server specific, FSAL specific
  and RPC Queue specific. For FSAL specific stats use option “fsal” and for
  RPC Queue specific statistics use "rpc".
- The second category is about managing the stats counting operations such as
  enabling/disabling/resetting etc.


To get short information on the usage of “ganesha_stats”, pass “help” argument
as shown below::

        # ganesha_stats help
        Command displays global stats by default.
        To display current status regarding stat counting use 
        /usr/bin/ganesha_stats status 
        To display stat counters use 
        /usr/bin/ganesha_stats [list_clients | deleg <ip address> | inode |
                                iov3 [export id] | iov4 [export id] | export |
                                total [export id] | fast | pnfs [export id] |
                                fsal <fsal name> | v3_full | v4_full | rpc | auth] 
        To reset stat counters use 
        /usr/bin/ganesha_stats reset 
        To enable/disable stat counters use 
        /usr/bin/ganesha_stats [enable | disable] [all | nfs | fsal | v3_full | v4_full | rpc |
                                auth]
        To get the current memory pool allocation
        /usr/bin/ganesha_stats pool


Options to extract stats
------------------------

- global – this is the default option, if no argument is passed. This option
    provides a global view of NFS server operation statistics related
    to different protocol versions.
- list_clients – this option provides information about all connected clients
    and also mentions whether per protocol are there any stats available or not
- deleg – this option provides information on delegations related to the
    specified client. This option needs additional argument specifying the
    client IP address. Delegations are not supported by NFS Ganesha on Spectrum
    Scale.
- inode – this option provides information on caching done by NFS Ganesha. The
    information includes cache hit, miss, conflict etc.
- iov3 – This option provides information on NFSv3 related stats. If NFSv3
    stats are available for an export, it provides details such as number
    requests, number of errors, latency etc.
- iov4 - This option provides information on NFSv4 related stats. If NFSv3
    stats are available for an export, it provides details such as number
    requests, number of errors, latency etc.
- export – this option provide information on the exports defined. Export id,
    export path, whether the export has any stats related to various protocols
    like NFSv3, NFSv4, NLM, 9P etc.
- total – this option provides information on all exports and provides number
    of operations done per protocol version (NFSV3, NFSv40, NFSv41 etc)
- fast – this option provides operations count per type of operation per
    protocol for the server.
- pnfs – this option provides NFSv4.1 specific statistics. NFSv4.1 is not
    supported by NFS Ganesha on Spectrum Scale.
- fsal – this option provides information about FSAL specific stats counting.
    This option requires additional parameter, which is FSAL name (e.g. gpfs).
    FSAL specific stats counting is disabled by default.
- v3_full – this option provides statistics of all NFSv3 operations. "v3_full"
    stats counting is disabled by default.
- v4_full – this option provides statistics of all NFSv4 operations (including
    all minor versions of NFSv4). "v4_full" stats counting is disabled by
    default.
- rpc – this option provides information about RPC queue related stats. RPC
    Queue specific stats counting is disabled by default.
- auth – this option provides information about authentication related stats
    for winbind and group cache. "auth" stats counting is disabled by default.
    This is available starting from Spectrum Scale 5.0.4 release.

Options to manage stats counting
--------------------------------

- help – this argument provides short help in “Ganesha_stats” utility.
- status – this option provides information about what all stats counting is
  enabled and since when. The different types of stats counting possible are
  nfs server, FSAL specific (fsal), v3_full, v4_full and RPC Queue (rpc)
- reset – this option helps in resetting all the stats counters (NFS serevr,
  FSAL specific, v3_full, v4_full & RPC Queue specific) to 0. This is helpful
  in case one wants to check the performance for a specific run.
- enable/disable – these 2 options helps in dynamically enabling or disabling
  stats counting. Both options requires additional argument specifying which
  stats counting needs to be enabled/disabled. The valid options are NFS server
  (nfs), FSAL specific (fsal), NFSv3 full (v3_full), NFSv4 full (v4_full), RPC
  Queue Specific (rpc) or all.
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
    a) # ganesha_stats fsal gpfs
    b) # ganesha_stats v3_full
    c) # ganesha_stats v4_full
    d) # ganesha_stats rpc
    e) # ganesha_stats reset  <= Make all counters 0


Few Examples:
=============

**Checking current status of various stats counting:**
::

    # ganesha_stats  status
    Stats counting for NFS server is enabled since:
            Thu Aug 23 00:40:19 2018658522215 nsecs
    Stats counting for FSAL is currently disabled
    Stats counting for RPC is currently disabled
    Stats counting for v3_full is currently disabled
    Stats counting for v4_full is currently disabled


**Enabling fsal specific stats:**
::

    # ganesha_stats  enable fsal
    Successfully enabled statistics counting


**Enabling RPC Queue specific stats:**
::

    # ganesha_stats  enable rpc
    Successfully enabled statistics counting

    # ganesha_stats  status
    Stats counting for NFS server is enabled since:
            Thu Aug 23 00:40:19 2018658522215 nsecs
    Stats counting for FSAL is enabled since:
            Thu Aug 23 00:43:53 2018326619720 nsecs
    Stats counting for RPC is enabled since:
            Thu Aug 23 00:44:12 201879056004 nsecs
    Stats counting for v3_full is currently disabled
    Stats counting for v4_full is currently disabled

*Additional examples:*
::

    # ganesha_stats  enable all
    Successfully enabled statistics counting

    # ganesha_stats status
    Stats counting for NFS server is enabled since: 
            Tue Jan 22 04:33:24 2019617715043 nsecs
    Stats counting for FSAL is enabled since: 
            Tue Jan 22 04:35:21 2019179381497 nsecs
    Stats counting for RPC is enabled since: 
            Tue Jan 22 04:35:24 2019255547453 nsecs
    Stats counting for v3_full is enabled since: 
            Tue Jan 22 04:37:09 2019256784851 nsecs
    Stats counting for v4_full is enabled since: 
            Tue Jan 22 04:37:09 2019256794253 nsecs


**Disabling stats (various examples):**
::

    # ganesha_stats  disable fsal
    Successfully disabled statistics counting

    # ganesha_stats  disable rpc
    Successfully disabled statistics counting

    # ganesha_stats status
    Stats counting for NFS server is enabled since: 
            Tue Jan 22 04:33:24 2019617715043 nsecs
    Stats counting for FSAL is currently disabled 
    Stats counting for RPC is currently disabled 
    Stats counting for v3_full is enabled since: 
            Tue Jan 22 04:37:09 2019256784851 nsecs
    Stats counting for v4_full is enabled since: 
            Tue Jan 22 04:37:09 2019256794253 nsecs

    # ganesha_stats  disable nfs
    Successfully disabled statistics counting

    # ganesha_stats status
    Stats counting for NFS server is currently disabled
    Stats counting for FSAL is currently disabled
    Stats counting for RPC is currently disabled
    Stats counting for v3_full is currently disabled
    Stats counting for v4_full is currently disabled


**Extracting protocol version specific stats:**
::

    # ganesha_stats v3_full
    NFSv3 Detailed statistics 
    Timestamp: Mon Jan 28 02:29:07 2019127096098 nsecs
    
    Operation Details                         |  Operation Latency                     |  Queue Latency
    ==========================================|========================================|=======================================
    Name            Total     Error      Dups |       Avg          Min           Max   |      Avg          Min           Max
    GETATTR            38         0         0 |     0.025362     0.010524     0.062460 |     0.018227     0.010014     0.026622
    SETATTR            10         0         0 |     1.288499     0.037150     5.801209 |     0.014868     0.009758     0.019687
    LOOKUP             16         0         0 |     0.065190     0.022066     0.134778 |     0.015255     0.009498     0.021964
    ACCESS             18         0         0 |     0.040911     0.012051     0.107734 |     0.018004     0.009885     0.042911
    READLINK            2         0         0 |     0.023475     0.023355     0.023596 |     0.017976     0.017347     0.018606
    READ                1         0         0 |     0.120613     0.120613     0.120613 |     0.010775     0.010775     0.010775
    WRITE               3         0         0 |     3.476515     0.943374     8.482333 |     0.015313     0.009988     0.019987
    CREATE              6         0         0 |    11.567501     0.875509    30.555752 |     0.017923     0.009620     0.036535
    REMOVE              5         0         0 |     1.005996     0.746162     1.141283 |     0.015108     0.009572     0.016710
    RENAME              1         0         0 |     2.538460     2.538460     2.538460 |     0.017815     0.017815     0.017815
    READDIRPLUS         5         0         0 |     0.281475     0.081980     0.905870 |     0.015604     0.011688     0.019417
    FSINFO              2         0         0 |     0.027047     0.012406     0.041689 |     0.010651     0.009803     0.011498
    PATHCONF            1         0         0 |     0.015112     0.015112     0.015112 |     0.010001     0.010001     0.010001

    # ganesha_stats v4_full
    NFSv4 Detailed statistics 
    Timestamp: Mon Jan 28 02:15:55 2019730861583 nsecs
    
    Operation Details                |  Operation Latency                     |  Queue Latency
    =================================|========================================|=======================================
    Name            Total     Error  |       Avg          Min           Max   |      Avg          Min           Max
    ACCESS              9         0 |     1.651072     0.020832    13.459373 |     0.020437     0.010946     0.061527
    CLOSE               8         0 |     0.062840     0.041341     0.133258 |     0.016127     0.011833     0.019749
    GETATTR           105         0 |     4.871099     0.020337   240.538668 |     0.017594     0.009267     0.061527
    GETFH              13         0 |    23.957723     0.037377   240.525346 |     0.014735     0.010946     0.017215
    LOOKUP              9         4 |     5.350111     0.035836    47.339675 |     0.016598     0.011479     0.028909
    OPEN               10         2 |    26.402119     0.071451   240.520741 |     0.015650     0.010946     0.017569
    OPEN_CONFIRM         1         0 |     0.043296     0.043296     0.043296 |     0.017917     0.017917     0.017917
    PUTFH             129         0 |     0.022270     0.011216     0.056945 |     0.017441     0.009267     0.061527
    READ                1         0 |     0.072095     0.072095     0.072095 |     0.009897     0.009897     0.009897
    READDIR             7         0 |    53.390574     0.049417   371.719512 |     0.018707     0.010025     0.039433
    READLINK            2         0 |     0.046855     0.045692     0.048018 |     0.014764     0.011391     0.018138
    REMOVE              5         0 |    36.401128     0.745786   146.347221 |     0.015334     0.011325     0.019788
    RENAME              1         0 |   180.708608   180.708608   180.708608 |     0.011648     0.011648     0.011648
    RENEW               2         0 |     0.024271     0.021181     0.027362 |     0.016085     0.015863     0.016308
    SAVEFH              1         0 |     0.024527     0.024527     0.024527 |     0.011648     0.011648     0.011648
    SETATTR             9         0 |    15.815532     0.969664    38.574486 |     0.016131     0.010168     0.020155
    WRITE               3         0 |    17.976211     8.583815    23.993364 |     0.014148     0.010053     0.017343


**Extracting fsal specific stats (considering fsal stats counting already enabled):**
::

    # ganesha_stats fsal gpfs
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

    # ganesha_stats rpc
    Timestamp: Mon Jan 28 02:16:51 2019898043588 nsecs
    
    RPC Receive Queue Statistics (4 Queues): 
        Pending Requests   :         0
        Completed Requests :       132
     QueueName       Pending  Complete      Avg wait      Max wait  (Milliseconds)
    REQ_Q_MOUNT                0         0      0.000000      0.000000
    REQ_Q_CALL                 0         0      0.000000      0.000000
    REQ_Q_LOW_LATENCY          0       107      0.015309      0.059700
    REQ_Q_HIGH_LATENCY         0        25      0.013215      0.023032
    
    
    RPC Send Queue Status (Milliseconds):
        RPCs sent -              132
        Avg wait time -     0.000405
        Max wait time -     0.000944

**Extracting Authentication related stats (assuming it is already enabled):**
::

    # ganesha_stats auth
    Timestamp: Thu May 16 13:41:58 2019870093583 nsecs
    Authentication related stats

    Group Cache
    Total ops: 4
    Ave Latency: 0.8792455
    Max Latency: 1.463887
    Min Latency: 0.093453

    Winbind
    Total ops: 44
    Ave Latency: 10.7604362955
    Max Latency: 206.015097
    Min Latency: 0.142931



**Resetting the stats:**
::

    # ganesha_stats  reset
    Successfully resetted statistics counters


