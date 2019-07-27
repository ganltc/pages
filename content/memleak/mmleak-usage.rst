===========================================================
Tracing nfs-ganesha daemon memory leaks with mmleak tool
===========================================================

:date: 2019-04-09
:modified: 2019-06-09
:tags: mmleak
:category: memory
:authors: Malahal Naineni

Tracing nfs-ganesha daemon memory leaks with mmleak tool
===========================================================

Valgrind and libasan are good tools to track memory leaks on test
systems and developer systems, but neither is a good tool to trace
memory leaks on production systems. The mtrace() [man 3 mtrace]
is better equipped to be run under production systems but unfortunately
they don't work with multi-threaded applications. I have seen people
using jemalloc but it seems to need some guess work.  More over, it
will use some memory for itself and uses a completely different
allocator.

https://github.com/malahal/mmleak is very similar to what mtrace does
but works with multi-threaded applications. Here are the instructions to
use mmleak with NFS-Ganesha daemon.

#. Download (git clone https://github.com/malahal/mmleak) mmleak project
   source code.
#. Run "make" to produce mmleak.so shared library for your architecture!
#. Copy mmleak.so, mmleak.py and mmleak-install scripts on the target
   system
#. Run "mmleak-install <dump-directory>". This will copy mmleak.so
   to /root and updates nfs-ganesha systemd service unit file to LD_PRELOAD
   the shared library.
#. Restart nfs service to make use of the tool::

    mmces service stop nfs && mmces service start nfs

#. mmleak.so stores its dump files into the directory given to
   mmleak-install script. mmleak-<HOSTNAME>.<PID>.pid is an active log
   file used by mmleak.so.  Other dump files matching
   mmleak-<HOSTNAME>.<PID>.<NUM>.out are complete and they can be
   copied to other systems for analysis.
#. The dump files generated will be big as it dumps all allocations and
   frees. Use mmleak-shrink.py script to remove the matching allocations and
   frees as below (**note that files with .pid extension are active dump
   files in use, please don't modify or use them**)::

    for i in mmleak*.out; do if [ ! -s $i.shrinked ]; then echo $i; mmleak-shrink.py < $i > $i.shrinked && rm $i; fi; done

#. See mmleak project for using the shrunken files and the process maps
   file to arrive at line numbers in the source code that allocated
   memory but didn't free.

#. To back out this LD_PRELOAD setup on the system::

    rm /etc/systemd/system/nfs-ganesha.service
    systemctl daemon-reload
    rm /root/mmleak.so # optional step!
    mmces service stop nfs && mmces service start nfs
