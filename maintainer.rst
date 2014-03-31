.. _maintainer:

Documentation for package maintainers
=====================================

Collecting more data for your package
-------------------------------------

If a program developer (or package maintainer) requires some specific
information which ABRT is not collecting, they can write a custom ABRT
hook which collects the required data for his program (package). Such
hook can be run at a different time during the problem processing
depending on how "fresh" the information has to be. It can be run:

1. at the time of the crash
2. at the time when user decides to analyse the problem (usually run ``gdb``
   on it)
3. at the time of reporting

All you have to do is create a ``.conf`` and place it to
``/etc/libreport/events.d/`` from this template::

     EVENT=<EVENT_TYPE> [CONDITIONS]
        <whatever command you like>

The commands will execute with the current directory set to the problem
directory (e.g: ``/var/spool/abrt/ccpp-2012-05-17-14:55:15-31664``)

If you need to collect the data at the time of the crash you need to
create a hook that will be run as a ``post-create`` event.

**WARNING: post-create events are run with root privileges!**

This sample hook will save the time to the file ``dateofcrash`` at the
time when the crash is detected by ABRT::

    EVENT=post-create component=binutils
            date > dateofcrash

A more realistic example: save the ``smart`` data when one of tools from
``udisks`` crashes::

    EVENT=post-create component=udisks
            which skdump >/dev/null 2>&1 || exit 0
            for f in /dev/[sh]d[a-z]; do
                test -e "$f" || continue
                skdump "$f"
                echo
            done >smart_data

If you want to collect the data at the time when the problem is being
reported, then you need to use a ``collect_`` event type. Example::

    EVENT=collect_syslog
            executable=`cat executable` &&
            base_executable=${executable##*/} &&
            grep -F -e "$base_executable" /var/log/messages | tail -999 >syslog

Testing how ABRT handles crashes of your application
----------------------------------------------------

To make sure abrt is able to handle the crash of your application
and generate backtraces correctly,
you can try to crash it by sending it a `SIGSEGV` signal
(only in case of C/C++ application)::

        kill -SIGSEGV $( pidof yourapp )

Abrt should catch the crash and produce a problem directory correctly.
It should be possible to report the crash to faf and bugzilla.

Using custom bugzilla template
------------------------------

ABRT allows package maintainers to override default Bugzilla bug
formatting for their packages by providing a component specific
template file for abrt. The file must be structured according to
:ref:`bz-plugin-configuration`
and must be stored in

- ``/etc/libreport/plugins/bugzilla_format_$component.conf`` for initial
  bug formatting and

- ``/etc/libreport/plugins/bugzilla_formatdup_$component.conf`` for
  additional bug comments

Template files references ABRT problem elements listed on :ref:`elements` page.

.. _bz-plugin-configuration:

Bugzilla plugin output configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Bugzilla reporter plugin accepts the ``-F`` option (default:
``/etc/libreport/plugins/bugzilla_format.conf``), which allows user to modify
the contents of the newly created bugs. Lines in this file have the following
format::

    # comment
    %summary:: summary format
    section:: element1[,element2]...
    the literal text line to be added to Bugzilla comment. Can be empty.

Summary format is a line of text, where ``%element%`` is replaced by text
element's content, and ``[[...%element%...]]`` block is used only if
``%element%`` exists. ``[[...]]`` blocks can nest.

Sections can be:

- ``%attach::`` a list of elements to attach.
- text, double colon (``::``) and a list of comma-separated elements.

Elements can be:

- problem directory element names, which get formatted as

  ::

      <element_name>: <contents>

  or

  ::

      <element_name>:
      :<contents>
      :<contents>
      :<contents>

- problem directory element names prefixed by ``%bare_``, which is formatted
  as-is, without ``<element_name>:`` and starting colons
- ``%oneline``, ``%multiline``, ``%text`` wildcards, which select all
  corresponding elements for output or attachment
- ``%binary`` wildcard, valid only for ``%attach`` section, instructs to attach
  binary elements
- problem directory element names prefixed by "-", which excludes given element
  from all wildcards

Nonexistent elements are silently ignored. If none of elements exists, the
section will not be created.

Example:

::

    %summary:: [abrt] %package%[[: %reason%]]

    This bug was automatically created by ABRT.

    Description of problem:: %bare_comment

    Version-Release number of selected component:: %bare_package
    Additional info:: \
     -analyzer,-count,-duphash,-uuid,-abrt_version,\
     -username,-hostname,\
     %oneline

    Bogosection:: nonexistent_element_name

    Backtrace:: %bare_backtrace

    %attach:: -comment,-reason,-reported_to,-event_log,%multiline,coredump

Note that empty lines are significant. In the above example, there is no
empty line between *Version-Release number of selected component* and
*Additional info* sections, which will result in these two sections
having no empty line between them in newly created bug's description
too.

Default configuration files
^^^^^^^^^^^^^^^^^^^^^^^^^^^

- initial bug formatting: https://github.com/abrt/libreport/blob/master/src/plugins/bugzilla_format.conf
- additional bug comments formatting: https://github.com/abrt/libreport/blob/master/src/plugins/bugzilla_formatdup.conf
