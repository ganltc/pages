============================================
Setting up environment to analyse coredumps
============================================

:date: 2019-08-29
:modified: 2019-08-29
:tags: coredump
:category: coredump
:authors: Malahal Naineni
:summary: Setting up environment to analyse coredumps

All we need is gdb with correct version of binaries and their debuginfo
rpms. If the coredump happens to be on RHEL7.4 and you have a RHEL7.3,
good luck installing correct binaries. Yum may bail out with broken
dependencies, and worse, if you provide --skip-broken, it may happily
install a version other than what you need. Of course, downgrading
packages is even worse!

Technically, you can install all the binary rpms (if you can download
them!) in a different location or just use rpm2cpio to get the shared
libraries and let gdb know where your shared libraries are. It is
possible this way, but docker seems a much better option! Here are the
steps I used. Bonus, if you mess up your docker container, spin a new
one from scratch!

1. Make sure your base system can run docker. Some older versions of
   RHEL7 have issues. I would start with RHEL7.5 or the latest and
   install docker.

2. SELinux didn't work with docker. There should be a way to get around
   this, but I didn't investigate it further. I just disabled SELinux in
   /etc/selinux/config file and rebooted the host system.

3. Download the coredump and other details on the docker host, e.g.
   at /data

4. Create a docker container with correct version as the system that took
   the coredump (Using RHEL7.4 here, power Linux docker image names
   might be different)::

        docker run -dt --name rh74 --hostname rh74 -v /data:/data:rw registry.access.redhat.com/rhel7.4
        Or for CentOS

        docker run -dt --name cos77 --hostname cos77 -v /data:/data:rw docker.io/library/centos:centos7.7.1908

5. Now execute bash in the running container as below::

        docker exec -it rh74 /bin/bash

6. You are in a bash shell! Use yum to install what ever rpms you need.
   The following yum install may fail, but you have a better chance of
   working as the docker container is close to the system that took
   coredump!::

        awk '{print $2}' dso_list > /tmp/rpms

        # edit /tmp/rpms to remove rpms that don't exist in yum repos.
        Also ganesha doesn't depend on glibc-common, so it won't be
        there in the dso_list file but will present issues with broken
        dependency. Adding the glibc-common with exact version as glibc
        needed to the /tmp/rpms will work!

        yum install $(cat /tmp/rpms)

7. Now run gdb and install any debuginfo rpms you need.

And if your docker container is stopped for some reason, you can always
restart it and then execute bash as below (nothing is lost)::

        docker start rh74
        docker exec -it rh74 /bin/bash
