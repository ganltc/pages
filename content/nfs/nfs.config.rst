===========================================
NFS Ganesha configuration on Spectrum Scale
===========================================

:date: 2018-02-14
:modified: 2020-03-10
:tags: nfs config
:category: nfs
:authors: Malahal Naineni
:summary: NFS Ganesha configuration on Spectrum Scale


Spectrum Scale protocol software provides a cool CLI tool (there is a
GUI as well!) to configure NFS Ganesha configuration. It actually
changes shared cluster configuration (to make sure all protocol nodes in
the cluster will have the same NFS Ganesha configuration) and restarts
or updates Ganesha daemon to make the changes effective. Careful as it
might restart Ganesha daemon. See **man mmnfs** for details.

Manual method
=============

Tools are good as long as they do what you want! Sometimes you may need
to go beyond a tool's capability! One example is that you want to change
an NFS Ganesha configuration parameter but that parameter is not
supported by mmnfs.  You have been warned that this is not an approved
procedure at the time of this writing. Make sure that the parameter you
are going to change is supported by the Ganesha daemon you are using!

1. Changing /var/mmfs/ces/nfs-config/gpfs.ganesha.export.conf
--------------------------------------------------------------

Luckily, you get a semi-automated way to modify the export configuration file!
Copy your existing export configuration file, preferably into /tmp, and
modify the way you want it. To save your export configuration into CCR
and restart NFS services automatically, execute the following command::

      mmnfs export load /tmp/gpfs.ganesha.export.conf


2. Changing /var/mmfs/ces/nfs-config/gpfs.ganesha.main.conf file
------------------------------------------------------------------

There is no "mmnfs config load <load-file>", likely due to "mmnfs
config" dealing with main.conf and log.conf, so we use "mmccr fput"
command to change the main configuration file.

As an example, the following steps can be used to change
**chunks_hwmark** parameter which tries to limit the total number of
chunk objects used in nfs-ganesha daemon. The default is 100,000 and the
following will change it to one million (1,000,000)

#. Find the section in the file this parameter belongs to.
   **chunks_hwmark** should be in **CacheInode** block.

#. Copy /var/mmfs/ces/nfs-config/gpfs.ganesha.main.conf file into /tmp
   and modify the /tmp/gpfs.ganesha.main.conf file. Add
   "Chunks_HWMark = 1000000;" to **CacheInode** block::

        CacheInode
        {
                fd_hwmark_percent=60;
                fd_lwmark_percent=20;
                fd_limit_percent=90;
                lru_run_interval=30;
                entries_hwmark=1500000;
                chunks_hwmark=1000000;
        }

#. Push the changed file to CCR::

       mmccr fput gpfs.ganesha.main.conf /tmp/gpfs.ganesha.main.conf

   This puts the last argument which should be a real file in your file
   system into CCR as gpfs.ganesha.main.conf file.
   
#. Above step should restart NFS Ganesha service on all protocol nodes.
   If this didn't happen for some reason, you can manually restart NFS
   service on all protocol nodes as below)::

       mmces service stop nfs -a && mmces service start nfs -a
