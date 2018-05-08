=========================================================
Create source code corresponding to a given rpm version
=========================================================

:date: 2018-05-08
:modified: 2018-05-08
:tags: source
:category: git
:authors: Malahal Naineni

Create source code corresponding to a given rpm version
=======================================================

#. Use git clone to clone the NFS-Ganesha software::

        git clone https://github.com/ganltc/nfs-ganesha.git

#. Go into the base directory of the git repo::

        cd nfs-ganesha

#. Use rpm command to get NFS-Ganesha package version, for example:

   On one system::

        # rpm -qa | grep nfs-ganesha
        nfs-ganesha-debuginfo-2.5.3-ibm015.03.el7.centos.x86_64
        nfs-ganesha-2.5.3-ibm015.03.el7.centos.x86_64
        nfs-ganesha-gpfs-2.5.3-ibm015.03.el7.centos.x86_64
        nfs-ganesha-utils-2.5.3-ibm015.03.el7.centos.x86_64

   On another system::

        # rpm -qa|grep nfs-ganesha
        nfs-ganesha-2.3.2-0.ibm53.el7.x86_64
        nfs-ganesha-gpfs-2.3.2-0.ibm53.el7.x86_64
        nfs-ganesha-utils-2.3.2-0.ibm53.el7.x86_64

#. The git tags that created the above packages should be **V2.5.3-ibm015.03**
   on the first system and **V2.3.2-ibm53** on the second system.

   *The first 3 version numbers (2.5.3 or 2.3.2) followed by the package name
   indicate that these packages are based on the upstream NFS-Ganesha
   version 2.5.3 and NFS-Ganesha version 2.3.2*

   *The string "-ibm" to all the way to ".el7" is added by IBM packaging.
   The tag is "Vx.y.z-<ibm-added-string>"*


   Find the exact tag from the git repo::
     
      $ git tag | grep ibm015.03
      V2.5.3-ibm015.03

#. Check out the above tag's source code with a local branch name same as the
   tag::

    git checkout -b V2.5.3-ibm015.03 V2.5.3-ibm015.03

#. Checkout libntirpc source code (used by NFS-Ganesha) as well::

    git submodule update --init

#. Now you should have the source code. All the source code is in "src"
   directory.
