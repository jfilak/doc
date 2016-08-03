.. _design:

Design
======

.. _abrtd:

Daemon
------

``abrtd`` is a daemon that watches for application crashes. When a crash occurs,
it collects the problem data (core file, application's command line, ...) and takes
action according to the configuration and the type of application that crashed.

By default it uses inotify interface [#inotify]_ to monitor the dump location
(``/var/tmp/abrt/``) for new directories created by C/C++ hook and a :ref:`socketapi`
(``/var/run/abrt/abrt.socket``) used by other hooks like :ref:`pyhook`.

The reason for using socket instead of direct filesystem access is security.
When a Python script throws unhandled exception, :ref:`pyhook` catches it, running
as a part of the broken Python application. The application is running
with certain SELinux privileges, for example it can not execute other
programs, or to create files in ``/var/tmp/abrt`` or anything else required
to properly fill a problem directory. Adding these privileges to every
application would weaken the security.
The most suitable solution for the Python application is
to open a socket where ``abrtd`` is listening, write all relevant
data to that socket, and close it. ``abrtd`` handles the rest of the processes.

.. _ccpphook:

C/C++ hook
----------

When C/C++ application crashes kernel uses :ref:`core_pattern` to
handle the crash. Abrt overrides default core_pattern with a pipe
to ``abrt-hook-ccpp`` executable that stores core dump in abrt's
dump location and notifies daemon about new crash. It also stores
number of files from ``/proc/<PID>/`` that might be useful
for debugging — ``maps``, ``limits``, ``cgroup``, ``status``.
Format and meaning of these files is described in the documentation
of the Linux kernel [#procfs]_.

To enable C/C++ hook use::

        systemctl abrt-ccpp start

.. _core_pattern:

core_pattern
^^^^^^^^^^^^

Variable used to specify a core dump file name template. If
the first character of the pattern is ``|``, the kernel will treat
the rest of the pattern as a command to run.  The core dump will be
written to the standard input of that program instead of to a file.

By default, ``/proc/sys/kernel/core_pattern`` contains ``core`` string
and kernel produces ``core.*`` files in crashed process` current directory.

Abrt's C/C++ hook overrides this with::

        |/usr/libexec/abrt-hook-ccpp %s %c %p %u %g %t e

which results in kernel calling ``abrt-hook-ccpp``. Detailed description
can be found in the documentation of the Linux kernel [#corepattern]_.

.. _debuginfo:

debuginfo
^^^^^^^^^

To be able to get full featured GDB backtrace from a core dump file, debuginfo
data must be available on the local file system. These data are usually
provided in the form of installable packages, however, ABRT needs to allow
non-privileged users to analyze the core dump file and report the
obtained backtrace to bug tracking tool. Hence, ABRT maintains its own
debuginfo directory ``/var/cache/abrt-di`` where all users can download and
unpack the required debuginfo packages through
``/usr/libexec/abrt-action-install-debuginfo-to-abrt-cache`` command line
utility.

Upon a new core dump file detection ABRT generates a list of build-ids
(``XXYYYY..YYYY``) using ``eu-unstrip -n --core=coredump``. When a user decides
to report the core dump file, the ABRT debuginfo tool goes through that list
and remembers those build-ids for which
``/usr/lib/debug/.build-id/XX/YYYY..YYYY.debug`` file does not exist in the
system root directory or in the ABRT debuginfo directory. Finally, packages
that provides the non-existing debug files are looked up in ``*debug*``
repositories and are downloaded and unpacked to the ABRT debuginfo directory.

.. _pyhook:

Python hook
-----------

Packages ``abrt-addon-python`` and ``abrt-addon-python3`` install
custom exception handler for Python 2 and Python 3 applications.

Python interpreter automatically imports ``abrt.pth`` file installed
in ``/usr/lib64/python2.7/site-packages/``. This file imports ``abrt_exception_handler.py``
that overrides Python's default ``sys.excepthook`` with custom handler
which forwards unhandled exceptions to ``abrtd`` via its :ref:`socketapi`.

Automatic import of site specific modules can be disabled by passing ``-S`` option
to python interpreter::

        python -S file.py


.. _eventdesign:

Events
------

A problem life cycle is driven by events in ABRT. For example:

* `Event 1` — a problem data directory is created.

* `Event 2` — problem data is analyzed.

* `Event 3` — a problem is reported to Bugzilla.

When a problem is detected and its defining data is stored,
the problem is processed by running events on the problem's data directory.
For event configuration how-to, refer to .

Standard ABRT installation currently supports several default
events that can be selected and used during problem reporting process.
Refer to :ref:`standardevents` to see the list of these events.

Only following three events are run automatically by ABRT:

``post-create``
        runs after the problem directory creation

``notify``
        runs after the processing chain is finished to notify user about new problem

``notify-dup``
        similar to ``notify`` for duplicate problems. See :ref:`dedup`.

.. _dedup:

Deduplication
-------------

When ABRT catches new crash it compares it to the rest of the stored problems
to avoid storing duplicate crashes.

It first checks if there is ``core_bactrace`` or ``uuid`` item in the problem
directory we are processing.

If there is a ``core_backtrace``, it iterates over all other dump
directories and computes similarity to their core backtraces (if any).
If one of them is similar enough to be considered duplicate, event processing
is stopped and only ``notify-dup`` event is fired.

If there is an ``uuid`` item (and no core backtrace), simple comparison
of ``uuid`` hashes is used for duplicate detection.

.. _elements:

Elements collected by ABRT
--------------------------

Commonly available elements:

===================== ======================================================== ====================
Property              Meaning                                                  Example
===================== ======================================================== ====================
``executable``        Executable path of the component which caused the        ``'/usr/bin/time'``
                      problem.  Used by the server to determine
                      ``component`` and ``package`` data.
``type``              Problem typem, see :ref:`problemtypes`.                  ``'Python'``
``component``         Component which caused this problem.                     ``'time'``
``hostname``          Hostname of the affected machine.                        ``'fiasco'``
``os_release``        Operating system release string.                         ``'Fedora release 17 (Beefy Miracle)'``
``uid``               User ID                                                  ``1000``
``username``          User name                                                ``'jeff'``
``architecture``      Machine architecture string                              ``'x86_64'``
``kernel``            Kernel version string                                    ``'3.6.6-1.fc17.x86_64'``
``package``           Package string                                           ``'time-1.7-40.fc17.x86_64'``
``time``              Time of the occurrence (unixtime)                         ``datetime.datetime(2012, 12, 2, 16, 18, 41)``
``count``             Number of times this problem occurred                     ``1``
``pkg_name``          Package name                                             ``'time'``
``pkg_epoch``         Package epoch                                            ``0``
``pkg_version``       Package version                                          ``'1.7'``
``pkg_release``       Package release                                          ``'40.fc17'``
``pkg_arch``          Package architecture                                     ``'x86_64'``
``uuid``              Unique problem identifier computed as a hash of the
                      first three frames of the backtrace                      ``'c55e3deb95d46553fdbefb1bc1d020e89a762fb7'``
===================== ======================================================== ====================

Elements dependent on problem type:

===================== ====================================================================== ====================================== ===============================
Property              Meaning                                                                Example                                Applicable
===================== ====================================================================== ====================================== ===============================
``abrt_version``      ABRT version string                                                    ``'2.0.18.84.g211c'``                  Crashes caught by ABRT
``cgroup``            cgroup (control group) information for crashed process                 ``'9:perf_event:/\n8:blkio:/\n...'``   C/C++
``core_backtrace``    Machine readable backtrace with no private data                                                               C/C++, Python, Ruby, Kerneloops
``backtrace``         Original backtrace or backtrace produced by retracing                                                         C/C++ (after retracing), Python, Ruby, Xorg, Kerneloops
                      process
``dso_list``          List of dynamic libraries loaded at the time of crash                                                         C/C++, Python
``exploitable``       Likely crash reason and exploitable rating                                                                    C/C++
``maps``              Copy of ``/proc/<pid>/maps`` file of the problem executable                                                   C/C++
``cmdline``           Copy of ``/proc/<pid>/cmdline`` file                                   ``'/usr/bin/gtk-builder-convert'``     C/C++, Python, Ruby, Kerneloops
``coredump``          Core dump of the crashing process                                                                              C/C++
``environ``           Runtime environment of the process                                                                            C/C++, Python
``open_fds``          List of file descriptors open at the time of crash                                                            C/C++
``pid``               Process ID                                                             ``'42'``                               C/C++, Python, Ruby
``proc_pid_status``   Copy of ``/proc/<pid>/status`` file                                                                           C/C++
``limits``            Copy of ``/proc/<pid>/limits`` file                                                                           C/C++
``var_log_messages``  Part of the ``/var/log/messages`` file which contains crash
                      information                                                                                                   C/C++
``suspend_stats``     Copy of ``/sys/kernel/debug/suspend_stats``                                                                   Kerneloops
``reported_to``       If the problem was already reported, this item contains                                                       Reported problems
                      URLs of the services where it was reported
``event_log``         ABRT event log                                                                                                Reported problems
``dmesg``             Copy of ``dmesg``                                                                                             Kerneloops
===================== ====================================================================== ====================================== ===============================


.. _problemtypes:

Supported problem types
^^^^^^^^^^^^^^^^^^^^^^^

Supported values for ``type`` element:

* ``CCpp``
* ``java``
* ``Kerneloops``
* ``selinux``
* ``Python``
* ``Python3``
* ``Ruby``
* ``xorg``

.. rubric:: Footnotes

.. [#procfs] http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/Documentation/filesystems/proc.txt
.. [#corepattern] http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/Documentation/sysctl/kernel.txt
.. [#inotify] http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/Documentation/filesystems/inotify.txt
