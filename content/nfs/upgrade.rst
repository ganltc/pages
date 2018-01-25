================================
NFS Ganesha upgrade instructions
================================

:date: 2018-01-25
:modified: 2018-01-25
:tags: nfs
:category: nfs
:authors: Malahal Naineni
:summary: NFS-Ganesha upgrade instructions


The following steps need to be carried out on each node where you want to
upgrade NFS Ganesha:

#. Issue the following command to suspend the node::

    mmces node suspend -N <Node>

#. Issue the following command to stop NFS services on the node::

    mmces service stop nfs -N <Node>

#. Issue the following command to upgrade NFS packages on the node::

    rpm -Uvh <all-needed-nfs-ganesha-rpms>

#. Issue the following command to start NFS services on the node::

    mmces service start nfs -N <Node>

#. Issue the following command to resume the node::

    mmces node resume -N <Node>
