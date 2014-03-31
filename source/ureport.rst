.. _ureport:

μReport
=======

μReport (`microreport`) is a JSON object representing a problem: binary crash, kerneloops, SELinux AVC denial...
These reports are designed to be small, machine readable and completely anonymous which allows us to use them for
automated reporting.
These reports allow us to keep track of bug occurrences while not necessarily
providing enough information for our engineers to fix the bug.
For that, you still may have to send a full bug report.

What exactly does a μReport contain?
------------------------------------

Each μReport generally contains a stack trace, or multiple stack traces in the case of multi-threaded C/C++ and Java programs.
The stack trace only describes the call stack of the program at the time of the crash and does not contain contents of any variables.

Every μReport also contains identification of the operating system,
versions of the RPM packages involved in the crash, and whether the program ran under a root user.

There are also items specific to each crash type:
- for C/C++ crashes, these are: path to the executable and signal delivered to the program
- for Python exceptions, there is the type of the exception (without the error message, which may contain sensitive data)
- for kernel oopses, these are: list of loaded kernel modules, list of taint flags, and full text of the kernel oops


Anonymity
---------
μReport **MUST NOT** contain any private data.
μReports are considered public and may be accessed through the web UI or web API,
forwarded to other instances or archived.
Clients need to think about the data they are sending.
These may contain passwords, credit card numbers, private keys etc.

Example:
Why not to include the message from python exception? Think about the following::

        ValueError: invalid literal for int() with base 10: 'my_secret_password'

Specification
-------------

The object contains a numeric field ``ureport_version``, which defines the expected contents.
If the field is missing, the version is considered 0.

μReport0 and former
^^^^^^^^^^^^^^^^^^^

First μReports were not versioned at all because of quick evolution.
μReport0 is the codename for the last of these (no known client has ever sent `ureport_version = 0`).
All former μReports will fail parsing. Although μReport0 is still supported (it can be turned into μReport1),
it is obsolete and should not be used in client applications.

μReport1
^^^^^^^^

μReport1 format was designed for the needs of ABRT (at that moment processing C/C++ crashes,
unhandled Python exceptions and kerneloopses).
The background idea is that all the reports contain a stacktrace and some metadata about the environment
(OS version, CPU architecture, kernel version...).
Only supported fields are allowed. Some of them are mandatory, some are not.
Any μReport containing excessive fields or missing mandatory fields will fail parsing.

Supported fields
""""""""""""""""

* ``ureport_version`` `(int)` MUST be set to ``1`` in order to be parsed and processed by μReport1 workflow
* ``type`` `(string)` either of ``userspace`` ``python`` ``kerneloops``
* ``reason`` `(string)` a short message describing what happend
* ``uptime`` `(int)` uptime of the machine
* ``component`` `(string)` affected component
* ``executable`` `(string)` affected executable
* ``architecture`` `(string)` CPU architecture
* ``crash_thread`` `(int)` ID of the crash thread
* ``kernel_taint_state`` `(string)` kernel tainted flags
* ``proc_status`` placeholder
* ``proc_limits`` placeholder
* ``oops`` `(string)` the raw kerneloops as written into syslog
* ``user_type`` `(string)` either of ``root`` ``nologin`` ``local`` ``remote``
* ``installed_package`` `(obj)` maps to appropriate RPM attributes

  * ``name`` `(string)`
  * ``epoch`` `(int)`
  * ``version`` `(string)`
  * ``release`` `(string)`
  * ``architecture`` `(string)`

* ``running_package`` `(obj)` same as ``installed_package``
* ``related_packages`` `(list)` list of following objects:

  * ``installed_package`` `(obj)` same as ``installed_package`` above
  * ``running_package`` `(obj)` same as ``installed_package`` above

* ``os`` `(obj)` operating system

  * ``name`` `(string)`
  * ``version`` `(string)`

* ``reporter`` `(obj)` identification of the client application

  * ``name`` `(string)`
  * ``version`` `(string)`

* ``core_backtrace`` `(list)` list of frames (following objects):

  * ``thread`` `(int)` thread number
  * ``frame`` `(int)` frame number
  * ``path`` `(string)` associated file
  * ``buildid`` `(string)` build-id of the file
  * ``offset`` `(int)` offset within the file
  * ``funcname`` `(string)` function name
  * ``funchash`` `(string)` function hash

* ``os`state`` `(obj)`

  * ``suspend`` `(string)` either ``yes`` or ``no``
  * ``boot`` `(string)` either ``yes`` or ``no``
  * ``login`` `(string)` either ``yes`` or ``no``
  * ``logout`` `(string)` either ``yes`` or ``no``
  * ``shutdown`` `(string)` either ``yes`` or ``no``

* ``selinux`` `(obj)`

  * ``mode`` `(string)` either of ``enforcing`` ``permissive`` ``disabled``
  * ``context`` `(string)` the affected context
  * ``policy`package`` `(obj)` same format as ``installed_package``

Example::

    {
            "ureport_version": 1,
            "type": "python",
            "reason": "TypeError",
            "uptime": 1,
            "component": "faf",
            "executable": "/usr/bin/faf-btserver-cgi",
            "installed_package": { "name": "faf",
                                   "version": "0.4",
                                   "release": "1.fc16",
                                   "epoch": 0,
                                   "architecture": "noarch" },
            "related_packages": [ { "installed_package": { "name": "python",
                                                           "version": "2.7.2",
                                                           "release": "4.fc16",
                                                           "epoch": 0,
                                                           "architecture": "x86_64" } } ],
            "os": { "name": "Fedora", "version": "16" },
            "architecture": "x86_64",
            "reporter": { "name": "abrt", "version": "2.0.7-2.fc16" },
            "crash_thread": 0,
            "core_backtrace": [
              { "thread": 0,
                "frame": 1,
                "buildid": "f76f656ab6e1b558fc78d0496f1960071565b0aa",
                "offset": 24,
                "path": "/usr/bin/faf-btserver-cgi",
                "funcname": "<module>" },
              { "thread": 0,
                "frame": 2,
                "buildid": "b07daccd370e885bf3d459984a4af09eb889360a",
                "offset": 190,
                "path": "/usr/lib64/python2.7/re.py",
                "funcname": "compile" },
              { "thread": 0,
                "frame": 3,
                "buildid": "b07daccd370e885bf3d459984a4af09eb889360a",
                "offset": 241,
                "path": "/usr/lib64/python2.7/re.py",
                "funcname": "_compile" }
            ],
            "user_type": "root",
            "selinux": { "mode": "permissive",
                         "context": "unconfined_u:unconfined_r:unconfined_t:s0",
                         "policy_package": { "name": "selinux-policy",
                                             "version": "3.10.0",
                                             "release": "2.fc16",
                                             "epoch": 0,
                                             "architecture": "noarch" } },
            "kernel_taint_state": "G    B      "
    }

Problems
""""""""

* Although μReport1 was designed to be independent on the operating system, it is closely related to Fedora.
* New types of problems are appearing, μReport1 is not generic enough to handle differences
  (SELinux AVC denial does not contain stacktrace, kerneloops does not contain executable...)
* All reports are handled in the same way. This either results into special-casing in code (``if type != "python"`` etc.)
  or into lower-quality results (clustering kerneloops and C/C++ with the same parameters does not work well).

μReport2
^^^^^^^^

The format of μReport2 is derived from the new pluginable model of FAF project. It only contains a common metadata and the identification of [OS plugin](https://github.com/abrt/faf/wiki/Operating-System-Plugins) and [Problem plugin](https://github.com/abrt/faf/wiki/Problem-Plugins). Only the common part is parsed by a global parser. `os` and `packages` are handled by OS plugin, `problem` is handled by Problem plugin. All excessive fields are ignored.

Supported fields
""""""""""""""""

* ``ureport_version`` `(int)` must be set to ``2`` in order to be parsed and processed by μReport2 workflow
* ``problem`` `(obj)` problem, passed to Problem plugin

  * ``type`` `(string)` must match to the ``name`` of any installed Problem plugin
  * anything else

* ``os`` `(obj)` operating system - passed to OS plugin

  * ``name`` `(string)` must match the ``name`` of any installed OS plugin
  * ``version`` `(string)` must match to the version in storage
  * ``arch`` `(string)` CPU architecture
  * anything else

* ``packages`` `(list)` list of affected packages, the nature of "package" is defined by the OS plugin
* ``reason`` `(string)` a short (human-readable) message describing what happend
* ``reporter`` `(obj)` identification of the client application

  * ``name`` `(string)`
  * ``version`` `(string)`

C/C++ crash example::

        {
          "ureport_version": 2,

          "reason": "Program /usr/bin/sleep was terminated by signal 11",

          "os": {
            "name": "fedora",                      # OS plugin
            "version": "18",
            "architecture": "x86_64"
          },

          "problem": {
            "type": "core",                        # problem plugin

            "executable": "/usr/bin/sleep",

            "signal": 11,

            "component": "coreutils",

            "user": {
              "local": true,
              "root": false
            },

            "stacktrace": [
              {
                "crash_thread": true,

                "frames": [
                  {
                    "build_id": "5f6632d75fd027f5b7b410787f3f06c6bf73eee6",
                    "build_id_offset": 767024,
                    "file_name": "/lib64/libc.so.6",
                    "address": 251315074096,
                    "fingerprint": "6c1eb9626919a2a5f6a4fc4c2edc9b21b33b7354",
                    "function_name": "__nanosleep"
                  },
                  {
                    "build_id": "cd379d3bb5d07c96d491712e41c34bcd06b2ce32",
                    "build_id_offset": 16567,
                    "file_name": "/usr/bin/sleep",
                    "address": 4210871,
                    "fingerprint": "d24433b82a2c751fc580f47154823e0bed641a54",
                    "function_name": "close_stdout"
                  },
                  {
                    "build_id": "cd379d3bb5d07c96d491712e41c34bcd06b2ce32",
                    "build_id_offset": 16202,
                    "file_name": "/usr/bin/sleep",
                    "address": 4210506,
                    "fingerprint": "562719fb960d1c4dbf30c04b3cff37c82acc3d2d",
                    "function_name": "close_stdout"
                  },
                  {
                    "build_id": "cd379d3bb5d07c96d491712e41c34bcd06b2ce32",
                    "build_id_offset": 6404,
                    "fingerprint": "2e8fb95adafe21d035b9bcb9993810fecf4be657",
                    "file_name": "/usr/bin/sleep",
                    "address": 4200708
                  },
                  {
                    "build_id": "5f6632d75fd027f5b7b410787f3f06c6bf73eee6",
                    "build_id_offset": 137733,
                    "file_name": "/lib64/libc.so.6",
                    "address": 251314444805,
                    "fingerprint": "075acda5d3230e115cf7c88597eaba416bdaa6bb",
                    "function_name": "__libc_start_main"
                  }
                ]
              }
            ]
          },

          "packages": [
            {
              "name": "coreutils",
              "epoch": 0,
              "version": "8.17",
              "architecture": "x86_64",
              "package_role": "affected",
              "release": "8.fc18",
              "install_time": 1371464601
            },
            {
              "name": "glibc",
              "epoch": 0,
              "version": "2.16",
              "architecture": "x86_64",
              "release": "31.fc18",
              "install_time": 1371464176
            },
            {
              "name": "glibc-common",
              "epoch": 0,
              "version": "2.16",
              "architecture": "x86_64",
              "release": "31.fc18",
              "install_time": 1371464184
            }
          ],

          "reporter": {
            "version": "0.3",
            "name": "satyr"
          }
        }

Python exception example::

        {
          "ureport_version": 2,

          "reason": "ImportError in /usr/bin/faf:55",

          "os": {
            "name": "fedora",                         # OS plugin
            "version": "18",
            "architecture": "x86_64"
          },

          "problem": {
            "type": "python",                         # problem plugin

            "component": "faf",

            "exception_name": "ImportError",

            "user": {
              "local": true,
              "root": false
            },

            "stacktrace": [
              {
                "file_name": "/usr/lib64/python2.7/site-packages/sqlalchemy/dialects/postgresql/psycopg2.py",
                "file_line": 312,
                "line_contents": "psycopg = __import__('psycopg2')",
                "function_name": "dbapi"
              },
              {
                "file_name": "/usr/lib64/python2.7/site-packages/sqlalchemy/engine/strategies.py",
                "file_line": 64,
                "line_contents": "dbapi = dialect_cls.dbapi(**dbapi_args)",
                "function_name": "create"
              },
              {
                "file_name": "/usr/lib64/python2.7/site-packages/sqlalchemy/engine/__init__.py",
                "file_line": 338,
                "line_contents": "return strategy.create(*args, **kwargs)",
                "function_name": "create_engine"
              },
              {
                "file_name": "/usr/lib/python2.7/site-packages/pyfaf/storage/__init__.py",
                "file_line": 213,
                "line_contents": "self._db = create_engine(config['storage.connectstring'])",
                "function_name": "__init__"
              },
              {
                "file_name": "/usr/lib/python2.7/site-packages/pyfaf/storage/__init__.py",
                "file_line": 199,
                "line_contents": "db = Database(debug=debug, dry=dry)",
                "function_name": "getDatabase"
              },
              {
                "file_name": "/usr/bin/faf",
                "file_line": 29,
                "line_contents": "db = getDatabase(debug=cmdline.sql_verbose, dry=cmdline.dry_run)",
                "function_name": "main"
              },
              {
                "file_name": "/usr/bin/faf",
                "file_line": 55,
                "special_function": "module",
                "line_contents": "main()"
              }
            ]
          },

          "packages": [
            {
              "name": "faf",
              "epoch": 0,
              "version": "0.9",
              "architecture": "noarch",
              "package_role": "affected",
              "release": "1.fc18"
            }
          ],

          "reporter": {
            "version": "0.3",
            "name": "satyr"
          }
        }

μReport attachment
^^^^^^^^^^^^^^^^^^

Currently used to attach Bugzilla ticket number to previously reported μReport identified by ``bthash``::

        {
          "type": "RHBZ",
          "bthash": "5f6632d75fd027f5b7b410787f3f06c6bf73eee6",
          "data": "123456"
        }
