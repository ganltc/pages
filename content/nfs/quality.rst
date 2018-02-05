===========================
Quality and data collection
===========================

:date: 2018-02-5
:modified: 2018-02-5
:tags: nfs
:category: nfs
:authors: Malahal Naineni
:summary: Quality and data collection


#. Many customers reported nfs-ganesha-lock.service failure.  This is
   normal due to the way we start CES and NFS services. We should fix
   this by bringing -H statd option to /etc/sysconfig/nfs on Redhat.
   **NFS/CES CLI work**. Need to investigate it on other distros.

#. Fix SELinux issue with Ganesha daemon placement or with a different
   directory (rename might be preferred). **Ganesha core, CES and NFS
   cli**

#. Adding Pseudo to CLI to hide file system tree AND to make security
   flavours work! NFV4 path traversal issues. Reported by few customers,
   it is an enhancement request. Ganesha core already supports it.
   **NFS CLI work**

#. Add some more data to snap collection. We need 'systemctl status'
   output among other things. Anything else that would be helpful?

#. Add Websphere MQ locks to our testing.
