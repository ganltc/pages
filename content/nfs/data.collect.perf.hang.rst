=========================================================
Data collection for NFS Ganesha hang or performance issue
=========================================================

:date: 2018-07-04
:modified: 2018-07-04
:tags: nfs
:category: nfs
:authors: Malahal Naineni
:summary: Data collection for NFS Ganesha hang or performance issue


If NFS clients become very slow or applications hang with files on NFS
mount, it is likely that the NFS server isn't responding or isn't
responding quick enough! An NFS server talks to various other software
components (authentication, back end file system etc), it is possible
that some other components might contribute to slowness but a hang is
something that is likely to be in NFS server itself. The following data
capture steps help identify if the observed problem is a performance
issue or a hang condition.

- Collect below data periodically on every CES node. These produce
  output, so please append the data to filenames of your choice. Adding
  a timestamp using "date" command is also preferred as given below for
  some commands (others have their own timestamp!).  These can be run
  every 5 minutes or less. This data collection should be started before
  the problem is observed.

  #. Extract GPFS stats and reset them for the next collection::

        mmfsadm vfsstats show  && mmfsadm vfsstats reset 

  #. Collect Ganesha stats::

        ganesha_stats

  #. Number of file descriptors opened by Ganesha server (**use the actual PID of Ganesha process**)::

        sh -c 'date && ls /proc/<PID-of-Ganesha-process>/fd | wc -l'

  #. Top 10 processes using the most amount of memory::

        sh -c 'date && ps aux --sort -rss | head'

  #. Free memory in the system::

        sh -c 'date && free -m'

- When the problem is observed with a CES node, collect this information on
  the CES node:

  #. Collect netstat output::

        sh -c 'date && netstat -an'

  #. Collect NFS-Ganesha kernel thread stacks::

        sh -c 'date && for i in /proc/<PID-of-Ganesha-process>/task/*; do echo "===$i===="; cat $i/stack; done'

  #. Enable Ganesha tracing (messages go to /var/log/ganesha.log)::

        ganesha_mgr set_log COMPONENT_ALL FULL_DEBUG

  #. Enable GPFS tracing::

        With at least vnode level 4 ??

  #. Collect kernel thread stacks::

        mmdumpkthreads

  #. Collect tcpdump at client & server for 5 minutes after enabling
     Ganesha and GPFS traces. Always collect tcpdump in pcap format by
     providing -w option to tcpdump command.

  #. Collect coredump by sending SIGABORT signal to ganesha process,
     make sure you setup your CES nodes to collect NFS-Ganesha coredumps
     first though. See `Setup to take ganesha coredumps
     <{filename}../coredump/coredump.rst>`_ for more details.
