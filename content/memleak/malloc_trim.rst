====================================================
Ganesha rpms with ARENA_MAX and malloc_trim support.
====================================================

:date: 2019-09-25
:modified: 2019-09-25
:tags: mmleak, trim
:category: memory
:authors: Malahal Naineni


glibc is known to use as many ARENAS (aka memory pools) as 8 times the
number of CPU threads a systems has. This makes a multi-threaded
application like NFS-Ganesha to use a lot more memory than what the
application actually needs. The following NFS-Ganesha rpms have settings
to limit MAX ARENAS as well as a dynamic malloc_trim() which release
memory back to the kernel. These are viewed as stop gap mechanisms until
NFS-Ganesha memory allocations are audited to manage long lived
allocations on its own!

- V2.3.2-ibm67  (for Scale version 4.2.3)
- V2.5.3-ibm036.10  (for Scale version 5.0.1 and up)
- V2.7.5-ibm052.00  (for Scale version 5.0.4 but should be usable with
  5.0.1 and up as well)

See https://ganltc.github.io/nfs-ganesha-upgrade-instructions.html for updating NFS-Ganesha packages

MALLOC_ARENA_MAX
================

The MALLOC_ARENA_MAX value from /etc/sysconfig/ganesha file is used
while starting up NFS-Ganesha daemon. If you want to restrict glibc to
use only 40 ARENAS, append the following line to /etc/sysconfig/ganesha
(yes, the string MALLOC_ARENA_MAX appears twice, outside the quotes and
inside the quotes!) and restart CES NFS service using mmces command::

    MALLOC_ARENA_MAX="MALLOC_ARENA_MAX=40"


Enable Dynamic Malloc Trim support
===================================

There are two ways to enable dynamic malloc_trim support.

1. This one changes ganesha config file
   /var/mmfs/ces/nfs-config/gpfs.ganesha.main.conf file. This method
   restarts nfs-ganesha daemon on all nodes due to mmccr update. See
   https://ganltc.github.io/nfs-ganesha-configuration-on-spectrum-scale.html
   for changing gpfs.ganesha.main.conf file! Add the following parameter
   to NFS_CORE_PARAM{} block to enable dynamic malloc trim support::

    enable_trim = true;

2. The second method uses the following dbus command to enable dynamic
   malloc trim support.  This needs to be run only when the nfs-ganesha
   daemon is already up and running. This affects only the node you run
   this command and you need to run every time you restart nfs-ganesha
   daemon as this is not persistent!::

	ganesha_mgr trim enable

Dynamic malloc_trim support commands
=====================================

1. To enable it (*disabled by default*)::

	ganesha_mgr trim enable

2. To disable it::

	ganesha_mgr trim disable

3. To know its status::

	ganesha_mgr trim status

4. To call malloc_trim() just one time::

	ganesha_mgr trim call
