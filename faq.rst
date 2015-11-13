.. _faq:

Frequently asked questions
==========================

Where does ABRT store the crashes?
----------------------------------

- ``/var/tmp/abrt``
- Prior to Fedora 18 these were stored in ``/var/spool/abrt``

What type of crashes can ABRT handle?
-------------------------------------

See :ref:`supported_langs`.

.. _wontcatch:

What to do when ABRT is not able to catch the crash of my application?
----------------------------------------------------------------------

- Make sure that following services are running:

  - ``abrtd``
  - ``abrt-ccpp``

  ``$ systemctl status abrtd && systemctl status abrt-ccpp``

- If one of them is not running you can use the following command to
  restart both of them:

  ``$ systemctl restart abrtd && systemctl status abrt-ccpp``

- If above doesn't help consult  ``journactl`` or ``/var/log/messages`` for error logs.

- By default ABRT won't handle crashes produced by 3rd party
  (unpackaged) software, for this to work read `How to enable handling
  of unpackaged software`_.

.. _unpackaged:

How to enable handling of unpackaged software
---------------------------------------------

- Edit ``/etc/abrt/abrt-action-save-package-data.conf`` and change
  ``ProcessUnpackaged = no`` to ``ProcessUnpackaged = yes``

  ``# sed -i 's/ProcessUnpackaged = no/ProcessUnpackaged = yes/' /etc/abrt/abrt-action-save-package-data.conf``

.. _nongpg:

How to enable handling of non-GPG signed software
-------------------------------------------------

- Edit ``/etc/abrt/abrt-action-save-package-data.conf`` and change
  ``OpenGPGCheck = yes`` to ``OpenGPGCheck = no``

  ``# sed -i 's/OpenGPGCheck = yes/OpenGPGCheck = no/' /etc/abrt/abrt-action-save-package-data.conf``

How do I list crashes handled by ABRT?
--------------------------------------

- Use either GUI application: ``$ gnome-abrt``

- or command line tool: ``$ abrt-cli list``

  See :ref:`usage` for more details.

What is μReport?
----------------

- μReport (`microreport`) is a JSON object representing a problem: binary
  crash, kerneloops, SELinux AVC denial, etc. These reports are designed
  to be small and completely anonymous which allows us to use them for
  automated reporting.

  See :ref:`ureport` page for more details.

What is tainted kernel and why is my kernel tainted?
----------------------------------------------------

The Linux kernel maintains a *taint state* which indicates whether
something happened to the running kernel that might caused a kernel
error.

Common reasons include:

- proprietary kernel module was loaded (P flag)
- previous kernel error (kerneloops) occurred (D flag).
- previous kernel warning (``GW`` flags).
- Both cases ``D`` flag and ``GW`` flags mean that kernel data structures may
  be corrupted. Therefore the current error is not necessary a real
  error, it could be a random consequence of the previous error.
- To get rid of the tainted kernel, you need to reboot your machine or
  stop loading proprietary modules.

- ABRT respects these flags and won't allow reporting if one or more
  are in effect because kernel developers are usually not able to fix
  issues when the kernel is tainted.

Complete list of taint flags::

  P - Proprietary module has been loaded.
  F - Module has been forcibly loaded.
  S - SMP with CPUs not designed for SMP.
  R - User forced a module unload.
  M - System experienced a machine check exception.
  B - System has hit bad_page.
  U - Userspace-defined naughtiness.
  D - Kernel has oopsed before
  A - ACPI table overridden.
  W - Taint on warning.
  C - modules from drivers/staging are loaded.
  I - Working around severe firmware bug.
  O - Out-of-tree module has been loaded.

Source:
http://git.kernel.org/?p=linux/kernel/git/torvalds/linux.git;a=blob;f=kernel/panic.c


How do I create a private bugzilla ticket?
------------------------------------------

ABRT can create reports with restricted access which means the
access to the report is limited to a group of trusted people. Please
note that the restriction differs between various bug trackers and even
if you mark something as restricted it still can leak to public, so if
you are not sure, then don't report anything!

To create a private bugzilla ticket, you have to specify the list of
groups to restrict the access to. The tricky part is that it has to be
the internal id of the group from bugzilla database. To ease the pain,
here is the list of the private group ids for supported bugzillas:

+------------------------------+------------------------------------------------------------+-----------------------------------+
| Bugzilla server              | group name                                                 | group ID to use in the settings   |
+==============================+============================================================+===================================+
| http://bugzilla.redhat.com   | Fedora Contrib (Bug accessible by Fedora Contrib members ) | fedora_contrib_private            |
+------------------------------+------------------------------------------------------------+-----------------------------------+
| http://bugzilla.redhat.com   | Private Group (Bug accessible only by the maintainer)      | private                           |
+------------------------------+------------------------------------------------------------+-----------------------------------+

How do I enable screencasting?
------------------------------

To enable screencasting in abrt you have to install fros package with
plugin matching your desktop environment. Currently there
are only 2 plugins available: ``fros-gnome`` and ``fros-recordmydesktop``. Gnome
plugin works only with Gnome 3, ``recordmydesktop`` should work with the most of
other desktop environments. To install the plugin run one of the
following commands (depending on your desktop environment)::

        yum install fros-gnome
        yum install fros-recordmydesktop

Why FAF collects tainted kernel oopses?
---------------------------------------

FAF collects tainted oopses because each received oops is forwarded 
to http://oops.kernel.org/ and kernel people want to
see **every** oops and **not only untainted** ones.

Why is my backtrace unusable?
-----------------------------
Unusable backtrace is usually caused by damaged core dump,
missing debug information or usage of unsupported coding technique 
(i.e. JavaScript in GNOME3).

These cause that the generated backtrace has low information value 
for developers because function names are replaced with ``'??'``
string which is place holder for unavailable function name.
In order to provide valuable crash reports,
ABRT will not let you create a Bugzilla bug for such a backtrace.

You can use ABRT to send the unusable backtrace to maintainers
via ``Email reporter``, but this is on your own responsibility.

.. _suided:

How to enable dumping of setuid binaries
----------------------------------------

By default kernel won't dump set-user-ID or otherwise protected/tainted binaries.
To change this behavior you need to change ``fs.suid_dumpable`` kernel variable.

To read the value use::

        sysctl fs.suid_dumpable


To change the value use::

        sysctl fs.suid_dumpable=0


Possible values are:

0. (`default`) — traditional behaviour. Any process which has changed
   privilege levels or is execute only will not be dumped.

1. (`debug`) — all processes dump core when possible. The core dump is
   owned by the current user and no security is applied. This is
   intended for system debugging situations only. Ptrace is unchecked.
   This is insecure as it allows regular users to examine the memory
   contents of privileged processes.

2. (`suidsafe`) — Any binary which normally would not be dumped (see "0"
   above) is dumped readable by root only. This allows the user to remove the
   core dump file but not to read it. For security reasons core dumps
   in this mode will not overwrite one another or other files. This  mode
   is appropriate when administrators are attempting to debug problems in a
   normal environment.

   Additionally, since Linux 3.6, /proc/sys/kernel/core_pattern must either be
   an absolute pathname or a pipe command, as detailed in core(5). Warnings
   will be written to the kernel log if core_pattern does not follow these
   rules, and no core dump will be produced.

Source:
http://man7.org/linux/man-pages/man5/proc.5.html


