=================
Building ganesha
=================

:date: 2019-08-22
:modified: 2019-08-22
:tags: nfs, build
:category: nfs
:authors: Malahal Naineni
:summary: Ganesha spectrum scale build instructions

Building ganesha
==================

Spectrum scale ganesha rpms are built from
https://github.com/ganltc/nfs-ganesha.git repository.  Branch "ibm2.7"
is based on ganesha V2.7. It is used for building spectrum scale 5.0.4
ganesha rpms.

Building ibm2.7 based rpms
--------------------------
1. Install rpms needed for building ganesha
    - RHEL7.x::

        yum groupinstall 'Development Tools'

        yum install cmake krb5-devel bison flex doxygen libuuid-devel \
        libblkid-devel libcap-devel dbus-devel python-devel libattr-devel \
        libnfsidmap-devel libwbclient-devel 

    - RHEL 8, apart from above, install these as well::

        yum install libnsl2-devel python3-devel selinux-policy-devel \
        libtirpc-devel

2. Install gpfs samba packages for AD kerberos support.
   Make sure you install Spectrum Scale Samba package (gpfs.smb) and its
   development package (gpfs.smb-devel). This step can be ignored if AD
   kerberos support is not needed.

3. git clone, checkout ibm2.7 and checkout submodules::

    git clone https://github.com/ganltc/nfs-ganesha.git
    cd nfs-ganesha
    git checkout <ibm2.7 or tag>
    git submodule update --init

4. Modify GANESHA_EXTRA_VERSION string in src/CMakeLists.txt file. This
   makes it clear that these rpms are privately built! For example
   having something like this:

   set(GANESHA_EXTRA_VERSION -ibm050.xyz)

5. Make a new build directory and run cmake from there::

    mkdir build && cd build && cmake ../src -DBUILD_CONFIG=rpmbuild \
    -DCMAKE_BUILD_TYPE=Release -DUSE_FSAL_GPFS=ON -DUSE_ADMIN_TOOLS=ON \
    -DUSE_GUI_ADMIN_TOOLS=OFF -DUSE_DBUS=ON
    
   Optionally, add "-D_MSPAC_SUPPORT=ON" for AD kerberos support!

6. Verify that cmake really did what you wanted! In particular look at
   its output and verify that it honored your features FSAL_GPFS,
   ADMIN_TOOLS, DBUS and MSPAC_SUPPORT etc.

7. Make source tarball for rpmbuild::

    make dist

8. Build rpms::

    QA_RPATHS=2 rpmbuild -ta nfs-ganesha*.tar.gz
