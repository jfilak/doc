.. _admin:


Documentation for administrators
================================

Configuring Centralized Crash Collection
----------------------------------------

In some scenarios, it's not desired to process crashes on affected machines.
In this case, ABRT offers upload functionality that can upload problems
caught by ABRT to remote machine via different protocols like scp, http or ftp.

Server side
^^^^^^^^^^^

To allow clients to upload problem directories to your server via scp
follow these steps:

1. Install ``abrt-addon-upload-watch``::

        yum install abrt-addon-upload-watch

2. Create ``abrt-upload`` user::

        useradd abrt-upload

3. Allow ``abrt-upload`` user write access to ``/var/spool/abrt-upload``::

        setfacl -m u:abrt-upload:-wx /var/spool/abrt-upload

4. Set password for ``abrt-upload`` user or generate a pair of keys to be used
   instead of a password. To generate authentication keys run::

        su - abrt-upload # become abrt-upload user
        ssh-keygen -f ~/.ssh/id_dsa -N '' # create new keypair with empty passphrase
        ln -s ~/.ssh/id_dsa.pub ~/.ssh/authorized_keys # allow loging in with newly created keys

4. In ``/etc/abrt/abrt.conf`` uncomment ``WatchCrashdumpArchiveDir = /var/spool/abrt-upload`` option.

5. Enable ``abrt-upload-watch.service``::

        systemctl start abrt-upload-watch.service
        systemctl enable abrt-upload-watch.service

Server is now ready to accept reports from clients.


Client side
^^^^^^^^^^^

On the client this functionality is provided by ``libreport-plugin-reportuploader``
package which contains ``reporter-upload`` executable.

Complete the following steps on every client system which will use centralized
crash reporting:

1. Install ``libreport-plugin-reportuploader``::

        yum install libreport-plugin-reportuploader

2. Modify the ``/etc/libreport/plugins/upload.conf`` configuration file so that
   the ``reporter-upload`` plugin knows where to copy the saved crash reports in the following way::

        URL = scp://USERNAME:PASSWORD@SERVERNAME/var/spool/abrt-upload/

   If you've chosen to use authentication keys instead of passwords,
   copy ``id_dsa`` file created in previous step to ``/root/id_dsa``.
   In this case your URL will look like this::

        URL = scp://USERNAME@SERVERNAME/var/spool/abrt-upload/

   If you're using passwords, make sure that ``/etc/libreport/plugins/upload.conf``
   is only readable by root::

        chmod 600 /etc/libreport/plugins/upload.conf

3. Allow automatic uploads::

        echo 'EVENT=notify reporter-upload' >> /etc/libreport/events.d/uploader_event.conf

4. Observe logs on both machines and try to crash something. You can either use ``will_segfault``
   from ``will_crash`` package or just crash the ``sleep`` binary with the following commands::

        sleep 100 &
        kill SIGSEGV %1


   If everything is configured correctly, problem directory should be transfered to server machine
   and the server should process it.


Troubleshooting
^^^^^^^^^^^^^^^

If uploading doesn't work make sure it's possible to upload problem directories manually
by running::

        reporter-upload -vvv -d /var/tmp/abrt/<PROBDIR>

Spacewalk
---------

Spacewalk is able to collect and report crash statistic from clients using abrt. More information and configuration
can be found on `spacewalk wiki <https://fedorahosted.org/spacewalk/wiki/HowToUseCrashReporting>`_.
