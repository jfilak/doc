.. _debugging:

Debugging ABRT
==============

Check steps described in :ref:`wontcatch`. If this won't
solve your problem proceed with steps described on this page.

ABRT log messages
-----------------

Look for failures in system log::

        journalctl | grep abrt

or if your system doesn't support journal::

        grep abrt /vart/log/messages

Possible reasons for abrt not handling crashes might include:
 * ``abrtd`` not running due to permission errors
 * ``DUP_OF_DIR: <dir>`` your crash is a duplicate of previous crash
 * ``Executable '<exe>' doesn't belong to any package and ProcessUnpackaged is set to 'no'`` — see :ref:`unpackaged`
 * Your package is not GPG signed — see :ref:`nongpg`
 * Crashing executable has setuid bit set — see :ref:`suided`


abrtd
-----

If ``abrtd`` malfunctions try stopping it and running it in foreground with verbose mode enabled::

        systemctl stop abrtd
        abrtd -vvv -d

If it hangs use ``gdb`` to attach to it and produce a backtrace::

        gdb /usr/sbin/abrtd $( pidof abrtd ) -batch -ex 'bt full'
