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
   cli work**

#. Automatically dump stack backtrace if Ganesha daemon crashes. This
   would help us quickly identify if it is a known issue or not. Many
   customers don't enable coredumps, so we end up asking them to
   recreate after setting up coredumps! **Ganesha core work**

#. Adding Pseudo to CLI to hide file system tree AND to make security
   flavours work! NFV4 path traversal issues. Reported by few customers,
   it is an enhancement request. Ganesha core already supports it.
   **NFS CLI work**

#. Preserve ENFILE or EMFILE errno's, now they show up as EIO!

#. Add some more data to snap collection. We need 'systemctl status'
   output among other things. Anything else that would be helpful?
   **snap collection work**

#. Add Websphere MQ locks to our testing. **Test**

#. ACL tests

#. Fix this build error::

       kernel: warning: `ganesha.nfsd' uses 32-bit capabilities (legacy support in use)

#. Spectrum documentation update
   #. Suggests "sync" mount option here::

         https://www.ibm.com/support/knowledgecenter/en/STXKQY_5.0.0/com.ibm.spectrum.scale.v5r00.doc/bl1adv_ces_nfssupport.htm
