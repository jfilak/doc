.. _interfacing:

Interfacing with ABRT
=====================

.. _socketapi:

Socket API
----------

Socket API allows creation of new problems. It is used by Python and Java hooks
to pass required data to ``abrtd``.

Socket path::

        /var/run/abrt/abrt.socket

First line has to contain HTTP header::

        POST / HTTP/1.1\r\n\r\n

followed by ``key=value`` pairs delimited with ``\0``.
The server expects another ``\0`` at the end of the message.

Mandatory keys:

* ``type`` — `(string)` problem type, see :ref:`problemtypes`.
* ``pid`` — `(integer)` 0 to `PID_MAX` (``/proc/sys/kernel/pid_max``)
* ``executable`` — `(string)` path of the affected executable
* ``backtrace`` — `(string)`
* ``reason`` — `(string)` reason of the crash

To ensure the problem can be reported to Bugzilla via report-gtk or
report-cli you have to add the following keys with the following contents:

* ``duphash`` - `(string)` duplicate hash, hash is placed in Bugzilla's
``Whiteboard`` field in format ``abrt_hash:$duphash``. Content of ``duphash``
is for C/C++ a sha1 of joined names of top 6 functions on the stacktrace. For
Python exception is a sha1 of the stacktrace.
* ``uuid`` - `(string)` local identifier of the problem. The content can be the
same as for ``duphash``.

Optionally, server accepts other elements listed in :ref:`elements`.

If there's no error server will respond with::

        HTTP/1.1 201 Created\r\n\r\n

or ``400`` status code in case of error.

:ref:`pyhook` may serve as an example of socket API usage.

.. _dbusapi:

DBus API
--------

Documentation for DBus API ``org.freedesktop.Problems`` is available
as part of ``abrt-dbus`` package or online
at http://jfilak.fedorapeople.org/ProblemsAPI/re01.html.

.. _pythonapi:

Python API
----------

Documentation for Python API is available in ``man abrt-python``
(part of ``abrt-python`` package) or online
at http://rmarko.fedorapeople.org/abrt-python/
