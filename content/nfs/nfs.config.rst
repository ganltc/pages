===========================================
NFS Ganesha configuration on Spectrum Scale
===========================================

:date: 2018-02-14
:modified: 2018-02-14
:tags: nfs config
:category: nfs
:authors: Malahal Naineni
:summary: NFS Ganesha configuration on Spectrum Scale


mmnfs
=====

Spectrum Scale protocol software provides a cool CLI tool (there is a
GUI as well!) to configure NFS Ganesha configuration. It actually
changes shared cluster configuration (to make sure all protocol nodes in
the cluster will have the same NFS Ganesha configuration) and restarts
or updates Ganesha daemon to make the changes effective. Careful as it
might restart Ganesha daemon. See **man mmnfs** for details.


Manual method
=============

Tools are good as long as they do what you want! Sometimes you may want
to go beyond a tool's capability! One example is that you want to change
an NFS Ganesha configuration parameter but that parameter is not
supported by mmnfs. This procedure is strictly not supported, so don't
blame anyone but yourself if things go wrong! :-) Again, you have been
warned that this is not an approved procedure at the time of this
writing.

Having said that, for example, the following steps can be used to change
**Dispatch_Max_Reqs** parameter which limits the number of total
outstanding NFS requests at Ganesha, anymore will be dropped:

#. Find the section and the file that parameter belongs to.
   **Dispatch_Max_Reqs** should be in **NFS_Core_Param** block which is
   in /var/mmfs/ces/nfs-config/gpfs.ganesha.main.conf file.

#. Add "Dispatch_Max_Reqs = 20000;" to **NFS_Core_Param** block::

        NFS_Core_Param
        {
            Nb_Worker = 256;
            Clustered = TRUE;
            NFS_Protocols= 3,4;
            NFS_Port = 2049;
            MNT_Port = 0;
            NLM_Port = 0;
            RQUOTA_Port = 0;
            RPC_Max_Connections = 10000;
            heartbeat_freq = 0;
            short_file_handle = FALSE;
            Dispatch_Max_Reqs = 20000;
        }


#. Push the changed file to shared cluster configuration::

       mmccr fput gpfs.ganesha.main.conf /var/mmfs/ces/nfs-config/gpfs.ganesha.main.conf

   This puts the last argument which should be a real file on your
   system into CCR as gpfs.ganesha.main.conf file.
   
#. Restart NFS Ganesha service on all protocol nodes::

       mmces service stop nfs -a && mmces service start nfs -a
