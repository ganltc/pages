===========================================
NFS Ganesha configuration on Spectrum Scale
===========================================

:date: 2018-02-14
:modified: 2018-02-14
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

Changing /var/mmfs/ces/nfs-config/gpfs.ganesha.export.conf
-----------------------------------------------------------

Luckily, you get a semi-automated way to modify the export configuration file!
Copy your existing export configuration file, preferably into /tmp, and
modify the way you want it. To save your export configuration into CCR
and restart NFS services automatically, execute the following command::

      mmnfs export load /tmp/gpfs.ganesha.export.conf


Changing /var/mmfs/ces/nfs-config/gpfs.ganesha.main.conf file
--------------------------------------------------------------

There is no "mmnfs config load <load-file>", likely due to "mmnfs
config" dealing with main.conf and log.conf, so we use "mmccr fput"
command to change the main configuration file.

As an example, the following steps can be used to change
**Dispatch_Max_Reqs** parameter which limits the number of total
outstanding NFS requests at Ganesha, anymore will be throttled!

#. Find the section and the file this parameter belongs to.
   **Dispatch_Max_Reqs** should be in **NFS_Core_Param** block which is
   in /var/mmfs/ces/nfs-config/gpfs.ganesha.main.conf file.

#. Copy /var/mmfs/ces/nfs-config/gpfs.ganesha.main.conf file into /tmp
   and modify the /tmp/gpfs.ganesha.main.conf file. Add
   "Dispatch_Max_Reqs = 20000;" to **NFS_Core_Param** block::

        NFS_Core_Param
        {
            Nb_Worker = 256;
            Clustered = TRUE;
            NFS_Protocols = 3,4;
            NFS_Port = 2049;
            MNT_Port = 0;
            NLM_Port = 0;
            RQUOTA_Port = 0
            RPC_Max_Connections = 10000;
            heartbeat_freq = 0;
            short_file_handle = FALSE;
            Dispatch_Max_Reqs = 20000;
        }


#. Push the changed file to shared cluster configuration::

       mmccr fput gpfs.ganesha.main.conf /tmp/gpfs.ganesha.main.conf

   This puts the last argument which should be a real file in your file
   system into CCR as gpfs.ganesha.main.conf file.
   
#. Restart NFS Ganesha service on all protocol nodes (this might not be
   needed)::

       mmces service stop nfs -a && mmces service start nfs -a
