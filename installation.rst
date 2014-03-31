.. _installation:

Installing ABRT
===============

.. _server_or_headless:

Server or headless machine
--------------------------

Use

``yum install abrt-cli``

to install basic support for crash collection
and set of command line tools for crash management and reporting.

If you want to receive email notifications about crashes caught by abrt you
need to install ``libreport-plugin-mailx`` package, which by default, sends
notifications to ``root`` at local machine. Email destination can be configured
in ``/etc/libreport/plugins/mailx.conf`` file.

If you prefer console notifications at login time,
install ``abrt-console-notification`` package. Notification sample::

        ABRT has detected 1 problem(s). For more info run: abrt-cli list --since 1394120788

Desktop
-------

Use

``yum install abrt-desktop``

to install complete desktop support including notification applet (``abrt-applet``)
and ``gnome-abrt`` — GUI for problem management and reporting.

From source
-----------

Clone the repositories you're interested in::

        git clone git@github.com:abrt/satyr.git
        git clone git@github.com:abrt/abrt.git
        git clone git@github.com:abrt/libreport.git
        git clone git@github.com:abrt/faf.git

Proceed with installing dependencies, you can get list of required
packages by running ``./autogen.sh sysdeps`` and directly install
these with ``yum`` by running ``./autogen.sh sysdeps --install``.

You can then build a project by running::

        ./autogen.sh
        ./configure
        make

If build passes continue with ``make rpm`` which will create
rpm packages in ``./build`` directory. This way is preferred
over ``make install`` installation as your package manager
keeps track of installed files allowing for easy removal.
