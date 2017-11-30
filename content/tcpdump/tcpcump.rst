================
tcpdump analysis
================

:date: 2017-09-16
:modified: 2017-09-16
:tags: tcpdump
:category: tcpdump
:authors: Malahal Naineni
:summary: tcpdump analysis for nfs issues

Hangs
=====

Note that if an NFS server doesn't respond to a client's request, the
request should be retried by the NFS client. Taking tcpdump for hangs
should be the best way to determine if it is a client's fault or the
server's fault. If you don't see anything in the tcpdump, then it is a
client's failure to retry. If the client is retrying but the server
isn't responding, then it is likely the server's issue. If they both
talk to each other same thing over and over again, then it is time to
consult the protocol to see why that is happening and whose fault it is.

Performance
===========

Performance should be analysed by performance counters. This is beyond
the scope of tmcpdump. Tcpdump sometimes gives a good indication of
performance issues by RTT statistics. The following pcap file shows high
WRITE (with FILE_SYNC mode) average response time (78ms), most likely
due to the backend file system being slow (with large files!). It most
likely caused other operations to be slow as well::

    $ tshark -q -zrpc,srt,100003,3  -r slow-writes.pcap

    =======================================================
    NFS Version 3 SRT Statistics:
    Filter: 
    Procedure        Calls    Min SRT    Max SRT    Avg SRT
    GETATTR           2549   0.000056   6.688285   0.222500
    SETATTR             79   0.000100   0.186188   0.008062
    ACCESS             541   0.014839   5.270222   0.285889
    READ                 2   0.132805   2.016926   1.074866
    WRITE             9188   0.000125   7.358130   0.784342
    READDIRPLUS       1821   0.000078   1.096325   0.042413
    FSSTAT            1210   0.000055   7.479788   0.322355
    COMMIT               1   0.022565   0.022565   0.022565
    =======================================================

Filter NFS port traffic
=======================

Port 2049 is used for NFSv3 and NFSv4. Note that MOUNT, NLM, NSM protocols used
in NFSv3 environments use random ports.

The following filter will display port 2049 (aka NFS) traffic only::

    $ tshark -Ynfs  -r nfs.pcap
    7          0 192.168.200.16 -> 192.168.200.184 NFS 4302 V3 WRITE Call, FH: 0xdb3717db Offset: 20603228160 Len: 4096 FILE_SYNC
    10          0 192.168.102.121 -> 192.168.102.183 NFS 210 V3 FSSTAT Call, FH: 0x6eb2ec3f
    14          0 192.168.200.27 -> 192.168.200.184 NFS 4310 V3 WRITE Call, FH: 0x50a17930 Offset: 2528481280 Len: 4096 FILE_SYNC
    27          0 192.168.101.41 -> 192.168.101.183 NFS 186 V3 GETATTR Call, FH: 0xc4cfb703
    31          0 192.168.200.16 -> 192.168.200.184 NFS 4302 V3 WRITE Call, FH: 0xc36dcd2e Offset: 21383094272 Len: 4096 FILE_SYNC
    35          0 192.168.200.11 -> 192.168.200.184 NFS 24790 V3 WRITE Call, FH: 0x4c45ddae Offset: 2464600064 Len: 24576 FILE_SYNC
    45          0 192.168.200.25 -> 192.168.200.184 NFS 8406 V3 WRITE Call, FH: 0xc0d7fbd3 Offset: 2523549696 Len: 8192 FILE_SYNC
    51          0 192.168.200.11 -> 192.168.200.184 NFS 3822 V3 WRITE Call, FH: 0x424ed19f Offset: 41099276288 Len: 53248 FILE_SYNC
    59          0 192.168.102.183 -> 192.168.102.121 NFS 230 V3 FSSTAT Reply (Call In 10)
    60          0 192.168.102.121 -> 192.168.102.183 NFS 210 V3 FSSTAT Call, FH: 0x68a15344
