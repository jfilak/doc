.. _supported_langs:

Supported programming languages and software projects
=====================================================

Overview
--------

========================== =======================
Langauge/Project           Package
========================== =======================
C                          abrt-addon-ccpp
C++                        abrt-addon-ccpp
Java                       abrt-java-connector
Python                     abrt-addon-python
Python 3                   abrt-addon-python3
Ruby                       rubygem-abrt
Linux (kernel oops)        abrt-addon-kerneloops
Linux (vmcore)             abrt-addon-vmcore
Linux (pstore)             abrt-addon-pstore
X.Org Server               abrt-addon-xorg
========================== =======================


C/C++
------

ABRT installs its own core dump handler via ``abrt-ccpp.service`` which when started,
overrides the default value of kernel's :ref:`core_pattern`. This causes
C/C++ crashes to be handled by ``abrtd`` and by default prevents creation
of ``core.*`` files in crashed process' current directory. More details available
in :ref:`ccpphook` design section.

Java
----

ABRT Java Connector is a JVM agent which reports uncaught Java exceptions to ABRT.
The agent registers several JVMTI event callbacks and has to be loaded into JVM using
``-agentlib`` command line parameter.

Python
------

Python hooks override the default ``sys.excepthook`` with custom function reporting
uncaught Python 2 and Python 3 exceptions to ``abrtd``.

Ruby
----

``rubygem-abrt`` registers a custom handler using ``at_exit`` feature executed when
the program ends which allows to check for possible unhandled exceptions.

Linux kernel
------------

Kernel oops
^^^^^^^^^^^

By checking the output of kernel logs, ABRT is able to catch and process so
called kernel oopses — non-fatal deviations from correct behavior of the Linux kernel.
This functionality is provided by ``abrt-addon-kerneloops`` and ``abrt-oops.service``.

Kernel panic
^^^^^^^^^^^^

ABRT is able to process ``vmcore`` files (kernel core dumps) produced on fatal
non-recoverable errors when kernel panics and reboot is required. If the
kernel crash dumping mechanism is enabled [#kdump]_ ``vmcore`` file
is produced at the time of the crash. ABRT is then able to read and process
the ``vmcore`` file from ``/var/crash/`` directory. This functionality
requires installing ``abrt-addon-vmcore`` and enabling ``abrt-vmcore.service``.

Persistent storage
^^^^^^^^^^^^^^^^^^

``abrt-addon-pstoreoops`` package contains plugin for collecting kernel
oopses from platform dependent persistent storage (pstore) [#pstore]_ [#pstore2]_.


.. rubric:: Footnotes

.. [#kdump] https://fedoraproject.org/wiki/How_to_use_kdump_to_debug_kernel_crashes
.. [#pstore] http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/Documentation/ABI/testing/pstore
.. [#pstore2] http://mjg59.dreamwidth.org/23554.html
