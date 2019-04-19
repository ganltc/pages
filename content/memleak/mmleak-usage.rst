===========================================================
Tracing nfs-ganesha daemon memory leaks with mmleak tool
===========================================================

:date: 2019-04-09
:modified: 2019-04-09
:tags: mmleak
:category: memory
:authors: Malahal Naineni

Tracing nfs-ganesha daemon memory leaks with mmleak tool
===========================================================

Valgrind and libasan are good tools to track memory leaks on test
systems and developer systems, but neither is a good tool to trace
memory leaks on production systems. The mtrace() [and mtrace perl] tool
is better equipped to be run under production systems but unfortunately
they don't work with multi-threaded applications. I have seen people
using jemalloc but it seems to need some guess work.  More over, it
will use some memory for itself and uses a completely different
allocator.

https://github.com/malahal/mmleak is very similar to what mtrace does
but works with multithreaded applications. Here are the instructions to
use mmleak with NFS-Ganesha daemon.

#. Download (git clone https://github.com/malahal/mmleak) mmleak project
   source code.
#. Run "make" to produce mmleak.so shared library for the right
   architecture!
#. Copy mmleak.so, mmleak.py and mmleak-install scripts on the target
   system
#. Run "mmleak-install <dump-directory>" script. This will copy mmleak.so
   to /root and updates nfs-ganesha systemd service unit file to LD_PRELOAD
   the shared library.
#. Restart nfs service to make use of the tool::

    mmces service stop nfs && mmces service start nfs

#. mmleak.so stores its dump files into the directory given to
   mmleak-install script. mmleak.out is an active log file used by
   mmleak.so.  Other dump files matching mmleak.<PID>.<NUM>.out are
   complete and they can be copied to other systems for analysis.
#. Completed dump files can be shrinked using the following command
   on each file (*the original dump file can be deleted after this*)::

    sort -s -k1,1 mmleak.<PID>.<NUM>.out | mmleak.py > mmleak.<PID>.<NUM>.out.shrinked

   If you have a bunch of dump files, you can run this loop::

        for i in mmleak.*.*.out; do if [ ! -s $i.shrinked ]; then echo $i; sort -s -k1,1 $i |/root/mmleak.py > $i.shrinked && rm $file; fi; done

#. See mmleak project for using the shrinked files and the process maps
   file to arrive at line numbers in the source code that allocated
   memory but didn't free.

#. To back out this LD_PRELOAD setup on the system::

        rm /etc/systemd/system/nfs-ganesha.service
        systemctl daemon-reload
        rm /root/mmleak.so # optional step!
        mmces service stop nfs && mmces service start nfs
