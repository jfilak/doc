.. _usage:

Usage
=====

Command line
------------

When crash is detected on a headless machine abrt
notifies user via email or console notification,
see :ref:`server_or_headless`.

To list all crashes on a machine run::

        abrt-cli list

Example output::

        id 0cf6d4371062e5fc386d242d4fbafd29f4ee8102
        Directory:      /var/tmp/abrt/ccpp-2013-12-27-13:55:34-24681
        count:          45
        executable:     /usr/bin/dzen2
        package:        dzen2-0.8.5-13.20100104svn.fc20
        time:           Fri 27 Dec 2013 01:55:34 PM CET
        uid:            1000

        id 57e6b30654ab84d516aae39ffa6fa266994fd633
        Directory:      /var/tmp/abrt/Python-2014-01-23-17:43:03-28776
        count:          1
        executable:     /usr/bin/gists
        package:        gists-0.4.5-4.fc20
        time:           Thu 23 Jan 2014 05:43:03 PM CET
        uid:            1000
        Reported:       https://retrace.fedoraproject.org/faf/reports/bthash/15cb1eb5b413a932201b1e7f53ac0abf0bc71e47
                        https://bugzilla.redhat.com/show_bug.cgi?id=1057230


Displayed are two crashes collected by abrt. Each crash has an identifier
and a directory that can be used for further manipulation using ``abrt-cli``.

To display detailed report about particular problem use::

        abrt-cli list -d <ID_OR_PATH>

To report a problem via ``abrt-cli`` use::

        abrt-cli report <ID_OR_PATH>

To delete a problem run::

        abrt-cli rm <ID_OR_PATH>

For more details consult ``man abrt-cli``.

Selecting a preferred text editor
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

During the reporting process ``abrt-cli report`` will
open a text editor. It uses the editor defined in
``$ABRT_EDITOR`` environment variable.
If the variable is not defined, it checks the
``$VISUAL`` and ``$EDITOR`` variables.
If none of these variables is set, ``vi`` is used.

You can set the preferred editor in your ``.bashrc``
configuration file. For example, if you prefer
GNU Emacs, add the following line to the file::

        export EDITOR=emacs

Graphical user interface
------------------------

After a crash is handled by ABRT user is presented
with a notification with options to ignore or report
the problem. If user chooses to report the problem
`gnome-abrt` application opens:

.. image:: img/gnome_abrt.png

`Report` button then starts a reporting wizard
guiding user through reporting process.

Testing ABRT functionality
--------------------------

To make sure you won't miss a crash of your application you
should verify that abrt works as expected.

Simplest way to do so is to crash ``sleep`` executable available
everywhere::

        sleep 10m &
        kill -SIGSEGV %1

Sleep then produces segmentation fault and should be caught
by abrt's C/C++ hook. If it's not working correctly consult
:ref:`debugging`.

will-crash
^^^^^^^^^^

For testing the functionality of various hooks we've created
a set of crashing executables called ``will-crash`` [#willcrash]_.

First, install the package::

        yum install will-crash

Then run one of the crashing executables depending on which
hook you would like to test, most commonly C/C++::

        will_segfault

This executable segfaults immediately and should be caught
by abrt. To get a list of other crashing executables run::

        rpm -ql will-crash | grep bin

.. rubric:: Footnotes

.. [#willcrash] http://github.com/sorki/will-crash
