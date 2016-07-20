.. _examples:

Examples
========

Automatic crash reporting through e-mails
-----------------------------------------

This example deals with configuration of libreport for sending full crash
reports including `backtrace` via an e-mail service from an autonomous system.
We usually encounter such systems in automatic testing environments. Let's
configure ABRT to send notification e-mails to `devel@lists.project.org`.

libreport's default configuration already supports sending of notification
e-mails to root user on a local system. This functionality is stored in
``/etc/libreport/events.d/mailx_event.conf`` file that contains the following
definition of ``notify`` and ``notify-dup`` events:

.. code:: bash

    EVENT=notify
        # do not rely on the default config nor on the config file
        Mailx_Subject="[abrt] a crash has been detected" \
        Mailx_EmailFrom="ABRT Daemon <DoNotReply>" \
        Mailx_EmailTo="root@localhost" \
        reporter-mailx --notify-only

    EVENT=notify-dup
        # do not rely on the default config nor on the config file
        Mailx_Subject="[abrt] a crash has been detected again" \
        Mailx_EmailFrom="ABRT Daemon <DoNotReply>" \
        Mailx_EmailTo="root@localhost" \
        reporter-mailx --notify-only

.. note:: The subject, sender and recipient are passed through the environment
  variables because `reporter-mailx` does not support these options in its
  command line arguments and putting it to its configuration file would break the
  general reporting workflow.

Detected problem types can be divided into two groups based on availability of
`backtrace` at the time of detection. The first group consists of Python,
Kernel oops, Ruby and Java problem types where `backtrace` is available at
the time of detection. The second group consists of CCpp (process crashes,
C/Cpp) where `backtrace` must be extracted from a dump file.

The default configuration of ``notify`` event is almost usable for the problem
types having backtrace available at the time of detection. We can reuse the
default configuration with some tweaks. We need to update recipient's address
and remove ``--notify-only`` string from `reporter-mailx`'s command line arguments
as it forces `reporters-mailx` to compose the message from a minimal subset of
problem data in order to not waste too much disk space on a local system where
all the excluded data are easily accessible anyway. We also need to configure
libreport to not use it for CCpp problems because we will provide its own event
definition.

The default ``notify-dup`` event is almost usable for all problem types because
it is run for consecutive occurrences of a single problem (``notify`` event is
run for the first occurrence and should create all the required data).  We only
need to update recipient's address because ``--notify-only`` argument make
sense for ``notify-dup`` event.

Since we are going to use `repoter-mailx` in several places and we do not want
to repeat ourself, we first move the static e-mail preferences (sender and
recipient) to ``/etc/libreport/plugins/mailx.conf``:

::

    EmailFrom="ABRT Daemon <DoNotReply>"
    EmailTo="devel@lists.project.org"


And the updated default configuration lines follow:

.. code:: bash

    EVENT=notify analyzer!=CCpp
        # full notify e-mail
        Mailx_Subject="[abrt] a crash has been detected" \
        reporter-mailx

    EVENT=notify-dup
        # full notify-dup e-mail
        Mailx_Subject="[abrt] a crash has been detected again" \
        reporter-mailx --notify-only

``"analyzer!=CCpp"`` tells libreport to run the lines above only if contents of
`analyzer` file does not equal to ``"CCpp"`` string.

We are done with the default configuration and now we are going to define
``notify`` event for CCpp. To be able to generate full GDB backtrace, we need
all relevant debug data. On Fedora like systems, ABRT can provide the required
debug data by extracting build ids from `coredump` file and unpacking rpm
packages providing paths made from the extracted build ids.

CCpp `notify` event lines follow:

.. code:: bash

    EVENT=notify analyzer=CCpp
        # CCpp notify event sending e-mail with full gdb backtrace
        # extract build ids from coredump file and save them in 'build_ids' file
        abrt-action-analyze-core --core=coredump -o build_ids >>event_log 2>&1 &&
        # download and unpack rpm package for each id in 'build_ids' file
        abrt-action-install-debuginfo -y >>event_log 2>&1 &&
        # call gdb to generate backtrace with local variables, call arguments
        # disassembly, etc. and save it in 'backtrace' file
        abrt-action-generate-backtrace >>event_log 2>&1 &&
        # generate 'duphash', 'crash_function' and 'backtrace_rating'
        abrt-action-analyze-backtrace >>event_log 2>&1
        #
        # send e-mail
        Mailx_Subject="[abrt] a crash has been detected" \
        reporter-mailx

Along with other problem data, the notification e-mail will also contain
`event_log` (useful for ABRT debugging), `backtrace_rating` (backtrace quality
measure value based on ration of resolved and unresolved frames) and
`duphash` which ABRT uses to identify a crash across all supported bug-tracking
systems and all machines (e.g. ABRT puts ``"abrt_hash:$(cat duphash)"`` string
to Whiteboard field of Bugzilla bug reports.)

Tip 1:
~~~~~~

You can add ``reporter-bugzilla -h $(cat duphash) >>bugzilla_bug_id`` command
to include existing Bugzilla bug report IDs for the currently processed crash.
This works on Fedora only.

Tip 2:
~~~~~~

You can make e-mail's subject more informative. The following script is not
bullet proof but produces really nice e-mail subject for C/Cpp problems:

.. code:: bash

    Mailx_Subject="[abrt] $(cat package || cat executable): $(cat crash_function && echo "():") $(cat reason || (cat executable && echo " crashed"))"

Getting core files from systemd-coredumctl
------------------------------------------

By default, ABRT detects crashes of native programs (C, C++) by its core dump
helper which is registered in ``/proc/sys/kernel/core_pattern``. Unfortunately,
there can be only one core pattern helper at the time so the ABRT core dump
helper cannot coexists with `systemd-coredumctl`.

However, the ABRT core dumper helper can be turned off and the ABRT
`systemd-coredumpctl` watcher can be used to make ABRT notified of crashes of
native programs.

Everything you need to do is to disable `abrt-ccpp.service` which replaces the
core_pattern configured via `sysctl` with the ABRT core pattern helper. If
the service is running the core_pattern should start with
``|/usr/libexec/abrt-hook-ccpp``.

.. code:: bash

    systemctl stop abrt-ccpp.service
    systemctl disable abrt-ccpp.service

Once you stop and disable the abrt-ccpp.service the core_pattern help should
start with ``|/usr/lib/systemd/systemd-coredump``. If it does not, please
check if the file ``/usr/lib/sysctl.d/50-coredump.conf`` exist and ensure that
there is no other file containing ``kernel.core_pattern=`` (sysctl.d(5)).

The last two things you need to do is to enable and start
`abrt-journal-core.service`.

.. code:: bash

    systemctl enable abrt-journal-core.service
    systemctl start abrt-journal-core.service

The current version of `abrt-journal-core.service` needs to make copies of data
that ABRT needs to be able to open a report in a bug tracking tool and leaves
the `systemd-coredumpctl` data untouched. That means that you cannot use the
ABRT journal core service to clean `systemd-coredumpctl` and you will end up
having two copies of core files, one in `systemd-coredumpctl`'s storage and one
in a sub-directory of ``/var/spool/abrt/``.

Collecting extra log files for crashes of a particular package
--------------------------------------------------------------

ABRT tries to provide people reading bug reports with as much information as
possible. Good source of problem details can be log files. Hence, upon
detection of a crashed process, ABRT goes through the system logs and copy lines
looking related to the crashed process to a file called ``var_log_messages`` in
the problem directory. The file can be found attached to bug reports opened by
ABRT/libreport.

However, application may not opt in for logging to system logs for various
reasons. In such case, ABRT can be configured to copy log files to problem
data directory and libreport will automatically attach them to bug reports.

Let's assume we have a package called ``foo`` and we want to copy log files
that are created in user's ``/var/run/`` directory. The application runs
several concurrent processes and each of the processes writes debug messages to
its own log file denoted by process' PID.

To get these log files, we need to create a new libreport EVENT configuration
file and instruct ABRT to run it after a crash of foo's executable appears. By
default, ABRT runs ``post-create``, ``notify``, and ``notify-dup`` EVENTs upon
new problem detection. We cannot use ``post-create`` because at that time
package of crashed executable might not be known. ``notify-dup`` is not
suitable because the event is executed for re-appearing problems, hence, the
log files should be already captured. Therefore we must define new ``notify``
EVENT.

.. code:: bash

    EVENT=notify pkg_name=foo
        # Copy log files of crashed process to foo.log
        cp /var/run/$(cat uid)/foo.$(cat pid).log foo.log

The ``pkg_name=foo`` string tells ABRT to run the lines above for problems in
any of executable shipped by the ``foo`` package. You can add as many such
conditions as you need. The lines below the first line are interpreted by
`/bin/sh` and current working directory contains all problem data captured by
ABRT.

The code must be placed in a file in the ``/etc/libreport/events.d/`` directory.
Packages often follow the ``${package name}_event.conf`` rule for these files.
