.. _howitworks:

How ABRT works
==============

When your application crashes, the crash event is handled
by one of ABRT's language or runtime dependent hooks which forwards
the crash information to :ref:`abrt daemon <abrtd>`. The daemon stores
the data in ``/var/tmp/abrt`` and runs a chain of events that
for example collect more data about the system, produce backtrace
from core dump or notifies a user. This process is followed by
automatic or manual reporting to various supported targets like bugzilla
or ABRT server (:ref:`faf`).

Reporting
---------

Currently, typical desktop reporting work-flow consists of generating so called
:ref:`ureport` (micro report) designed to be completely anonymous so it can be sent
and processed automatically avoiding costly bugzilla queries and manpower.

The ABRT server in this scenario works like a first line of defense — collecting
massive amounts of similar reports and responding with tracker URLs
in case of known problems.

If user is lucky enough to hit a unique issue not known by faf,
reporting chain continues with reporting to bugzilla, more complex process
which requires user having a bugzilla account and going through numerous steps
including verification that the report doesn't contain sensitive data.

ABRT also features unattended reporting possibilities in form of email reports,
upload functionality and integration with tools like Spacewalk or Foreman
for large scale deployment scenarios.


.. unused
        .. graphviz::

           digraph foo {
              rankdir = "LR";
              "Application crash" -> "Hook(s)";
              "Hook(s)" -> "Daemon";
              "Daemon" -> "Dump directory";
              "Daemon" -> "Events";
              "Events" -> "Dump directory";
              "Dump directory" -> "Reporting" [style=dotted];
           }


Components
----------

.. _abrt:

abrt
""""

The daemon and collection of tools for handling crashes and
monitoring logs for errors. Takes care of storing and processing
of the crashes caught by various hooks.

.. _gnome-abrt:

gnome-abrt
""""""""""

Simple GUI application for problem management and reporting.

.. image:: img/gnome_abrt.png

.. _libreport:

libreport
"""""""""

Libraries providing an API for reporting problems
via different paths like email, bugzilla, faf, scp upload..

.. _faf:

faf
"""

Crash collecting server, also known as ABRT server. Provides
accurate statistics of incoming reports and acts as a proxy in front
of bugzilla (or any other issue tracker) when it comes to
automatic reporting of crashes. It's designed to receive
anonymous :ref:`μReports <ureport>` and to find clusters of similar reports
among them. For reports that are known, user receives fast response
containing links to faf's problem page, issue tracker or an entry
from knowledge base, that contains number of well-known issues like
usage of proprietary kernel modules or browser crashes caused by
unsupported modules.

.. _satyr:

satyr
"""""

Satyr is a collection of low-level algorithms for program failure processing,
analysis, and reporting supporting kernel space, user space, Python, and Java
programs.  Considering failure processing, it allows to parse failure
description from various sources such as GDB-created stack traces, Python stack
traces with a description of uncaught exception, and kernel oops message.
Infromation can also be extracted from the core dumps of unexpectedly
terminated user space processes and from the machine executable code of
binaries.  Considering failure analysis, the stack traces of failed processes
can be normalized, trimmed, and compared.  Clusters of similar stack traces can
be calculated.  In multi-threaded stack traces, the threads that caused the
failure can be discovered.  Considering failure reporting, the library can
generate a failure report in a well-specified format, and the report can be
sent to a remote machine.

.. _retrace_server:

retrace server
""""""""""""""

The retrace server provides a core dump analysis and backtrace
generation service over a network using HTTP protocol. Its functionality
is currently being merged to faf.

Privacy
-------

:ref:`μReports <ureport>` consist of structured data
suitable to be analyzed in a fully automated manner, though
they do not necessarily contain sufficient information to fix
the underlying problem. The reports are designed not to
contain any potentially sensitive data to eliminate the need
for review before submission.
