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
