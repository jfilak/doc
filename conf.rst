.. _configuration:

Configuration
=============

Configuration files
-------------------

Upon installation, ABRT and libreport place their respective configuration files into the several directories on a system:

``/etc/libreport/``
    contains the ``report_event.conf`` main configuration file. More information about this configuration file can be found in :ref:`eventconf`.

``/etc/libreport/events/``
    holds files specifying the default setting of predefined events.

``/etc/libreport/events.d/``
    keeps configuration files defining events.

``/etc/libreport/plugins/``
    contains configuration files of programs that take part in events.

``/etc/abrt/``
    holds ABRT specific configuration files used to modify the behavior of ABRT's services and programs. More information about certain specific configuration files can be found in :ref:`abrtspecific`.

``/etc/abrt/plugins/``
    keeps configuration files used to override the default setting of ABRT's services and programs. For more information on some specific configuration files refer to :ref:`abrtspecific`.


.. _abrtspecific:

ABRT Specific Configuration
---------------------------

Standard ABRT installation currently provides the following ABRT specific configuration files:

``/etc/abrt/abrt.conf``
    allows you to modify the behavior of the abrtd service.

``/etc/abrt/abrt-action-save-package-data.conf``
    allows you to modify the behavior of the abrt-action-save-package-data program.

``/etc/abrt/plugins/CCpp.conf``
    allows you to modify the behavior of ABRT's core catching hook.

The following configuration directives are supported in the ``/etc/abrt/abrt.conf`` file:

* ``WatchCrashdumpArchiveDir = /var/spool/abrt-upload``

This directive is commented out by default. Enable it if you want `abrtd` to auto-unpack crashdump tarball archives (`.tar.gz`)
which are located in the specified directory. In the example above, it is the ``/var/spool/abrt-upload/`` directory.
Whichever directory you specify in this directive, you must ensure that it exists and it is writable for abrtd.
The ABRT daemon will not create it automatically.
If you change the default value of this option, be aware that in order to ensure proper functionality of ABRT,
this directory **must not** be the same as the directory specified for the ``DumpLocation`` option.

.. caution::
        Do not modify this option when using SELinux

        Changing the location for crashdump archives will cause SELinux denials unless you reflect the change in respective SELinux rules first.
        See the `abrt_selinux(8)` manual page for more information on running ABRT in SELinux.

Remember that if you enable this option when using SELinux, you need to execute the following command in order to set the
appropriate SELinux boolean allowing ABRT to write into the ``public_content_rw_t`` domain::

     setsebool -P abrt_anon_write 1

* ``MaxCrashReportsSize = size_in_megabytes``

This option sets the amount of storage space, in megabytes, used by ABRT to store all problem information from all users.
The default setting is 1000 MB. Once the quota specified here has been met, ABRT will continue catching problems,
and in order to make room for the new crash dumps, it will delete the oldest and largest ones.

* ``DumpLocation = /var/spool/abrt``

This directive is commented out by default. It specifies the location where problem data directories are created and in which
problem core dumps and all other problem data are stored.
The default location is set to the ``/var/tmp/abrt`` directory.
Whichever directory you specify in this directive, you must ensure that it exists and it is writable for `abrtd`.
If you change the default value of this option, be aware that in order to ensure proper functionality of ABRT,
this directory **must not** be the same as the directory specified for the ``WatchCrashdumpArchiveDir`` option.

.. caution::
        Do not modify this option when using SELinux

        Changing the dump location will cause SELinux denials unless you reflect the change in respective SELinux rules first.
        See the `abrt_selinux(8)` manual page for more information on running ABRT in SELinux.

Remember that if you enable this option when using SELinux, you need to execute the following command in order to set the appropriate
SELinux boolean allowing ABRT to write into the ``public_content_rw_t`` domain::

     setsebool -P abrt_anon_write 1

The following configuration directives are supported in the ``/etc/abrt/abrt-action-save-package-data.conf`` file:

* ``OpenGPGCheck = yes/no``

Setting the OpenGPGCheck directive to yes (the default setting) tells ABRT to only analyze and handle crashes in applications provided by packages which are signed by the GPG keys whose locations are listed in the /etc/abrt/gpg_keys file. Setting OpenGPGCheck to no tells ABRT to catch crashes in all programs. 

* ``BlackList = nspluginwrapper, valgrind, strace, [more_packages ]``

Crashes in packages and binaries listed after the BlackList directive will not be handled by ABRT. If you want ABRT to ignore other packages and binaries, list them here separated by commas. 

* ``ProcessUnpackaged = yes/no``

This directive tells ABRT whether to process crashes in executables that do not belong to any package. The default setting is `no`.

* ``BlackListedPaths = /usr/share/doc/*, */example*``

Crashes in executables in these paths will be ignored by ABRT.

The following configuration directives are supported in the ``/etc/abrt/plugins/CCpp.conf`` file:

* ``MakeCompatCore = yes/no``

This directive specifies whether ABRT's core catching hook should create a core file, as it could be done if ABRT would not be installed.
The core file is typically created in the current directory of the crashed program but only if the ``ulimit -c`` setting allows it.
The directive is set to `yes` by default.

* ``SaveBinaryImage = yes/no``

This directive specifies whether ABRT's core catching hook should save a binary image to a core dump.
It is useful when debugging crashes which occurred in binaries that were deleted. The default setting is `no`.

Configuring ABRT to Detect a Kernel Panic
-----------------------------------------

ABRT can detect a kernel panic using the ``abrt-vmcore`` service, which is provided by the ``abrt-addon-vmcore`` package.
The service starts automatically on system boot and searches for a core dump file in the ``/var/crash/`` directory.
If a core dump file is found, ``abrt-vmcore`` creates the problem data directory in the ``/var/tmp/abrt/``
directory and moves the core dump file to the newly created problem data directory.
After the ``/var/crash/`` directory is searched through, the service is stopped until the next system boot.

To configure ABRT to detect a kernel panic, perform the following steps:

#. Ensure that the ``kdump`` service is enabled on the system.
   Especially, the amount of memory that is reserved for the ``kdump`` kernel has to be set correctly.
   You can set it by using the ``system-config-kdump`` graphical tool, or by specifying the
   ``crashkernel`` parameter in the list of kernel options in the ``/etc/grub2.conf`` configuration file.

#. Install and enable the ``abrt-addon-vmcore`` package using yum::

     yum install abrt-addon-vmcore
     systemctl enable abrt-vmcore

   This installs the ``abrt-vmcore`` service with respective support and configuration files.

#. Reboot the system for the changes to take effect.

Unless ABRT is configured differently, problem data for any detected kernel panic is now stored
in the ``/var/tmp/abrt/`` directory and can be further processed by ABRT just as any other detected kernel oops.


Desktop Session Autoreporting
-----------------------------

Enabled autoreporting behavior
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

With the desktop session autoreporting enabled, ABRT automatically uploads
:ref:`uReport` for user problems immediately upon their detection. If the abrt
server (:ref:`faf`) knows the reported problem, the server provides additional
information about the problem and ABRT informs the user that the detected
problem is known via a notification bubble. The notification bubble offers
showing the problem's web page, opening the problem in the ABRT GUI or simply
ignoring the problem. If the problem is unknown, ABRT shows a notification
bubble and the user can either start reporting process as usual or ignore the
problem.

Disabled autoreporting behavior
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

With the autoreporting disabled, ABRT uploads an :ref:`uReport` for the
detected problem after click on *"Report" button* in the notification bubble. If
the detected problem is not known to abrt server, ABRT proceeds
with reporting wizard.

Event though ABRT notifies about problems of other users, it never uploads
uReports for these problems automatically. The other user's problems are always
processed in the way of processing problems with the autorepoting disabled,
which is described in the 2nd paragraph.

In case of unavailable network, ABRT will postpone notification of the detected
problems until the network becomes available again. The list of postponed
problems will be held only for a single user desktop session. The postponed
problems may not be notified at all, if the network won't become available
during the desktop session's lifetime.

Uploading uReports requires a writeable problem directory and in order to make
the reporting more automatic and less confusing, ABRT might move the problem
directory from the system dump location (usually ``/var/tmp/abrt/`` directory) to
``$HOME/.cache/abrt/spool/`` directory without asking the user for a permission to do
so. However, ABRT moves the directories only if the user has no rights for writing
to the system dump location.

Enabling desktop session autoreporting
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The desktop autoreporting can be enabled in various ways. The easiest way is to answer
**Yes** in a dialogue asking for *enabling automatically submitted crash reports* which
appears after clicking on *"Report" button* in the notification bubble. The second way is
to open the ``Automatic Bug Reporting Tool`` application,
open the application menu and click on the following option::

  ABRT Configuration

and turn on option::

  Automatically send uReport

The last but the most inconvenient way is to manually edit file::

  $HOME/.config/abrt/settings/abrt-applet.conf

and add the following line::

  AutoreportingEnabled = yes

System Autoreporting
--------------------

ABRT can be configured to submit an :ref:`uReport` for each of the detected problems to the
abrt server (:ref:`faf`) immediately upon their detection. The server provides
the following information about the submitted problem:

- URLs to existing bug reports if any (Bugzilla bugs)
- short description text

System Autoreporting can be enabled by issuing the following command::

  abrt-auto-reporting enabled

or via Augeas_::

  augtool set /files/etc/abrt/abrt.conf/AutoreportingEnabled yes

or by adding the following line to the ``/etc/abrt/abrt.conf`` configuration file::

  AutoreportingEnabled yes

When System Autoreporting is enabled, Desktop Session Autoreporting is enabled
too.

.. _Augeas: http://augeas.net/

Shortened Reporting
-------------------

Enables shortened GUI reporting where the reporting is interrupted
after AutoreportingEvent is done. It means that the reporting is done
when user clicks *"Report" button* on the notification buble. Upon that,
ABRT uploads an uReport_ describing handled problem, shows
a notification bubble stating that the problem has been reported and finishes.

Shortened Reporting has no effect on the reporting process started from
the GUI, because we wanted to allow advanced users to easily submit
full bug report into Bugzilla. We believe that all users who care about
detected crashes and open **Automatic Bug Reporting Tool** application to see
them are advanced users.

  ``Default value: Yes but only if application is running in GNOME desktop``

To turn Shortened Reporting on open::

  Automatic Bug Reporting Tool

go to the application menu and click::

  ABRT Configuration

and turn on option::

  Shortened Reporting

Or manually edit file::

  $HOME/.config/abrt/settings/abrt-applet.conf

and add there the following line::

  ShortenedReporting = yes

.. _uReport: https://github.com/abrt/faf/wiki/uReport

Automatic sensitive data filtering
----------------------------------

ABRT keeps the global list of *sensitive words* in
``/etc/libreport/forbidden_words.conf`` so in order to change this list for
all users, system administrator has to edit this file.  There is also per-user
list in ``$HOME/.config/abrt/settings/forbidden_words.conf`` (doesn't exist
by default, so user has to create it). The format of the file is one word per
line. Wildcards are **NOT** supported.

The forbidden words are sometimes a part of other words and these are usually
not deemed as sensitive information. Offering such false positive
sensitive words for review by user makes the process of removing sensitive
data from reports hard and the real sensitive data may be missed. Therefore, ABRT
has another list of words that are never considered as sensitive information.
The list contains common words consisting from *the sensitive words*. The global
list of *ignored words* is kept in file:

  ``/etc/libreport/ignored_words.conf``

And the per-user list:

  ``$HOME/.config/abrt/settings/ignored_words.conf``

.. _eventconf:

Event configuration
-------------------

Each event is defined by one rule structure in a respective configuration file.
The configuration files are typically stored in the ``/etc/libreport/events.d/``
directory. These configuration files are loaded by the main configuration file,
``/etc/libreport/report_event.conf``.

The ``/etc/libreport/report_event.conf`` file consists of include directives
and rules. Rules are typically stored in other configuration files in
the ``/etc/libreport/events.d/`` directory.

If you would like to modify this file,
please note that it respects shell metacharacters (``*``, ``$``, ``?``, etc.)
and interprets relative paths relatively to its location.

Each rule starts with a line with a non-space leading character,
all subsequent lines starting with the space character or the tab character
are considered a part of this rule.
Each rule consists of two parts, a condition part and a program part.
The condition part contains conditions in one of the following forms::

    VAR=VAL,

    VAR!=VAL

    VAL~=REGEX

where:

* ``VAR`` is either the ``EVENT`` key word or a name of a problem data directory
  element such as ``executable``, ``package``, ``hostname``, ... See :ref:`elements`
  for more.

* ``VAL`` is either a name of an event or a problem data element, and

* ``REGEX`` is a regular expression.

The program part consists of program names and shell interpretable code.
If all conditions in the condition part are valid,
the program part is run in the shell. The following is an event example::

        EVENT=post-create date > /tmp/dt
                echo $HOSTNAME `uname -r`

This event would overwrite the contents of the ``/tmp/dt`` file with
the current date and time, and print the hostname of the machine
and its kernel version on the standard output.

Here is an example of a yet more complex event which is actually
one of the predefined events.
It saves relevant lines from the ``~/.xsession-errors`` file to
the problem report for any problem for which the ``abrt-ccpp``
services has been used to process that problem,
and the crashed application has loaded any `X11` libraries at the time of crash::

        EVENT=analyze_xsession_errors analyzer=CCpp dso_list~=.*/libX11.*
                test -f ~/.xsession-errors || { echo "No ~/.xsession-errors"; exit 1; }
                test -r ~/.xsession-errors || { echo "Can't read ~/.xsession-errors"; exit 1; }
                executable=`cat executable` &&
                base_executable=${executable##*/} &&
                grep -F -e "$base_executable" ~/.xsession-errors | tail -999 >xsession_errors &&
                echo "Element 'xsession_errors' saved"

The set of possible events is not hard-set.
System administrators can add events according to their need.
Currently, the following event names are provided with standard ABRT and libreport installation:

``post-create``
    This event is run automatically by ``abrtd`` on newly created
    problem data directories.
    When the ``post-create`` event is run, ``abrtd`` checks whether
    the ``UUID`` identifier of the new problem data matches the ``UUID``
    of any already existing problem directories.
    If such a problem directory exists, the new problem data is deleted.
    See :ref:`dedup` for more details on duplicate handling.

``analyze_name_suffix``
    where `name_suffix` is the adjustable part of the event name.
    This event is used to process collected data.
    For example, the ``analyze_LocalGDB`` runs the GNU Debugger (``GDB``)
    utility on a core dump of an application and produces a backtrace of
    a crash.

``collect_name_suffix``
    where name_suffix is the adjustable part of the event name.
    This event is used to collect additional information on a problem.

``report_name_suffix``
    where name_suffix is the adjustable part of the event name.
    This event is used to report a problem.



Additional information about events (such as their description,
names and types of parameters which can be passed to them as
environment variables, and other properties) is stored in
the ``/etc/libreport/events/event_name.xml`` files.
These files are used by both GUI and CLI to make the user interface more friendly.
Do not edit these files unless you want to modify the standard installation.

.. _standardevents:

Standard ABRT Installation Supported Events
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Standard ABRT installation currently provides a number of default analyzing,
collecting and reporting events. Some of these events are configurable
using the ``gnome-abrt`` GUI application.
The following is a list of default analyzing, collecting and reporting
events provided by the standard installation of ABRT:

``analyze_VMcore`` — Analyze VM core
    Runs ``GDB`` (the GNU debugger) on problem data of an application
    and generates a backtrace of the kernel.
    It is defined in the ``/etc/libreport/events.d/vmcore_event.conf``
    configuration file.

``analyze_LocalGDB`` — Local GNU Debugger
    Runs ``GDB`` (the GNU debugger) on problem data of an application
    and generates a backtrace of a crash.
    It is defined in the ``/etc/libreport/events.d/ccpp_event.conf``
    configuration file.

``analyze_RetraceServer`` — Generate backtrace remotely
    Uploads core dump to :ref:`retrace_server` for remote backtrace generation.
    It is defined in the ``/etc/libreport/events.d/ccpp_retrace_event.conf``
    configuration file.

``analyze_xsession_errors`` — Collect .xsession-errors
    Saves relevant lines from the ``~/.xsession-errors`` file to the problem report.
    It is defined in the ``/etc/libreport/events.d/ccpp_event.conf``
    configuration file.

``report_Logger`` — Logger
    Creates a problem report and saves it to a specified local file.
    It is defined in the ``/etc/libreport/events.d/print_event.conf``
    configuration file.

``report_RHTSupport`` — Red Hat Customer Support
    Reports problems to the Red Hat Technical Support system.
    This possibility is intended for users of Red Hat Enterprise Linux.
    It is defined in the ``/etc/libreport/events.d/rhtsupport_event.conf``
    configuration file.

``report_Mailx`` — Mailx
    Sends a problem report via the ``mailx`` utility to a specified email address.i
    It is defined in the ``/etc/libreport/events.d/mailx_event.conf``
    configuration file.

``report_Uploader`` — Report uploader
    Uploads a tarball (`.tar.gz`) archive with problem data to the chosen
    destination using the FTP or the SCP protocol.
    It is defined in the ``/etc/libreport/events.d/uploader_event.conf``
    configuration file.

``report_uReport`` — :ref:`ureport` uploader
    Uploads :ref:`ureport` to :ref:`faf` server.

``report_EmergencyAnalysis`` — Upload problem directory to :ref:`faf`
    Uploads a tarball to :ref:`faf` server for further analysis. Used
    in case of reporting failure when standard reporting methods fail.
    It is defined in the ``/etc/libreport/events.d/emergencyanalysis_event.conf``
    configuration file.

Adjusting plugin configuration
------------------------------

ABRT reports problems to various destinations. Almost every reporting
destination require some configuration. For instance, Bugzilla requires
login and password and URL to an instance of the Bugzilla service. Some
configuration details can have default values (e.g. Bugzilla's URL) but
others don't have sensible defaults (e.g. login).

ABRT lets user provide configuration through text configuration files, such as
``/etc/libreport/events/report_Bugzilla.conf``. All text configuration files
consist of key/value pairs.

The event text configuration can be stored in one of these files:

- ``/etc/libreport/events/somename.conf`` - for system scope configuration
- ``$XDG_CACHE_HOME/abrt/events/somename.conf`` - for user scope configuration [XDG_]

.. _XDG: http://standards.freedesktop.org/basedir-spec/basedir-spec-latest.html

These files are the bare minimum necessary for running events on the
problem directories. ABRT GUI and CLI tools will read configuration data
from these files and pass it down to events they run.

However, in order to make GUI interface more user-friendly, additional
information can be supplied in XML files in the same directory, such as
``report_Bugzilla.xml``. These files can contain the following information:

- user-friendly event name and description (*Bugzilla*, *Report to Bugzilla bug tracker*).
- the list of items in problem directory which are required for event to succeed.
- default and mandatory selection of items to send or not send.
- whether GUI should prompt for data review.
- what configuration options exist, their type (string, boolean, etc), default value, prompt string, etc. This lets GUI to build the appropriate configuration dialogs.

ABRT's GUI saves configuration options in ``gnome-keyring`` or ``ksecrets`` and
passes them down to events, overriding data from text configuration files.

You can obtain a set of keys for a particular event by executing of the
following command::

    xmllint --xpath "/event/options/option/@name" $EVENT_XML_FILE | sed 's/name="\([^ ]*\)"/\1\n/g'

The mapping between event XML definition files and event configuration
files:

+-----------------------------------------------+---------------------------------+----------------------------------+
| Event name                                    | Definition file                 | Configuration file               |
+===============================================+=================================+==================================+
| Bugzilla                                      | report\_Bugzilla.xml            | report\_Bugzilla.conf            |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Logger                                        | report\_Logger.xml              | report\_Logger.conf              |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Analyze C/C++ Crash                           | analyze\_CCpp.xml               | analyze\_CCpp.conf               |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Local GNU Debugger                            | analyze\_LocalGDB.xml           | analyze\_LocalGDB.conf           |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Retrace Server                                | analyze\_RetraceServer.xml      | analyze\_RetraceServer.conf      |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Analyze VM core                               | analyze\_VMcore.xml             | analyze\_VMcore.conf             |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Collect GConf configuration                   | collect\_GConf.xml              | collect\_GConf.conf              |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Collect Smolt profile                         | collect\_Smolt.xml              | collect\_Smolt.conf              |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Collect system-wide vim configuration files   | collect\_vimrc\_system.xml      | collect\_vimrc\_system.conf      |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Collect your vim configuration files          | collect\_vimrc\_user.xml        | collect\_vimrc\_user.conf        |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Collect .xsession-errors                      | collect\_xsession\_errors.xml   | collect\_xsession\_errors.conf   |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Post report                                   | post\_report.xml                | post\_report.conf                |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Kerneloops.org                                | report\_Kerneloops.xml          | report\_Kerneloops.conf          |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Mailx                                         | report\_Mailx.xml               | report\_Mailx.conf               |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Red Hat Customer Support                      | report\_RHTSupport.xml          | report\_RHTSupport.conf          |
+-----------------------------------------------+---------------------------------+----------------------------------+
| Report uploader                               | report\_Uploader.xml            | report\_Uploader.conf            |
+-----------------------------------------------+---------------------------------+----------------------------------+
| uReport                                       | report\_uReport.xml             | report\_uReport.conf             |
+-----------------------------------------------+---------------------------------+----------------------------------+

By default the ABRT complains about missing configuration if any of mandatory
options is not configured. Mandatory option is option not marked as 'Allow
empty'. Run the following command to obtain the list of mandatory options::

    xmllint --xpath "/event/options/option[allow-empty!='yes']/@name" $EVENT_XML_FILE \
            | sed 's/name="\([^ ]*\)"/\1\n/g'
