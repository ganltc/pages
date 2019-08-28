===============================
Setup to take ganesha coredumps
===============================

:date: 2017-09-22
:modified: 2017-12-13
:tags: coredump
:category: coredump
:authors: Malahal Naineni
:summary: Setup to take ganesha coredumps


Redhat RHEL7.x
==============

Automatic Bug Reporting Tool (**ABRT**) is the preferred method to take
application coredumps on Redhat Enterprise Linux systems. The following
steps can be used to configure an RHEL 7.x system to take Ganesha
coredumps:

#. Install the abrt-cli RPM if not already installed::

        yum install abrt-cli

#. IBM ganesha package are not signed, so you need to configure ABRT to
   take coredumps from executables belonging to unsigned packages::

        Set OpenGPGCheck=no in the /etc/abrt/abrt-action-save-package-data.conf file.

#. Allow coredump to be unlimited in size (limited by the space
   available in /var/spool/abrt filesystem)::

        Set MaxCrashReportsSize = 0 in the /etc/abrt/abrt.conf file.

#. Start (or restart) the abrt daemon service::

        systemctl restart abrtd

#. Start (or restart) the abort-ccpp service::

        systemctl restart abrt-ccpp


#. A core dump might not be generated for code areas where the Ganesha
   process changed its credentials. To take coredump from all paths,
   fs.suid_dumpable should be set to 2::

        Insert the following entry into the /etc/sysctl.conf file:
        fs.suid_dumpable = 2

        Run the following to make the above setting effective:
        sysctl -p

#. Verify that kernel.core_pattern is set to abrt-hook-ccpp (something
   like ``|/usr/libexec/abrt-hook-ccpp %s %c %p %u %g %t e %P %I %h``
   and fs.suid_dumpable is set to 2::

        # sysctl kernel.core_pattern
        kernel.core_pattern = |/usr/libexec/abrt-hook-ccpp %s %c %p %u %g %t e %P %I %h

        # sysctl fs.suid_dumpable
        fs.suid_dumpable = 2

#. Verify that the system actually takes Ganesha coredumps when it
   crashes! Here we send abort signal to Ganesha process to
   intentionally crash it, please note that the Ganesha daemon will be
   terminated after this and the generated coredump should be deleted as it is
   useless::

        # kill -SIGABRT <Ganesh-daemon-PID>

For additional details, see the Documentation about ABRT-specific configuration.
