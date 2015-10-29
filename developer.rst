.. _developer:

Documentation for developers
============================

Contributing
------------

Send us patches to our mailing list::

        http://lists.fedorahosted.org/mailman/listinfo/crash-catcher

or create a pull request for the corresponding repository::

        https://github.com/abrt


Where appropriate, your submission should come with a test —
either unit test or as a part of our integration test suite.
See :ref:`newinttest` for more details.


Nightly builds
--------------

Nightly builds and repositories for Fedora and RHEL
are avaialable at http://rmarko.fedorapeople.org/

Ignoring common functions on the stack
--------------------------------------

To improve clustering of similar crashes done by
:ref:`faf`, backtraces are first normalized to skip
common functions like ``_start`` from glibc or
``__kernel_vsyscall`` from Linux kernel.

Such functions are listed in
`satyr/lib/normalize.c <https://github.com/abrt/satyr/blob/master/lib/normalize.c>`_ file.

Writing man pages
-----------------

Man pages file can be written in AsciiDoc and then translated into the classic
man page format. Man pages in AsciiDoc format are stored in doc/ directory. For example
for libreport are placed in libreport/doc/.

To beter understand the issue, the following links shows man page written in Ascii::

    http://www.methods.co.nz/asciidoc/asciidoc.1.txt

The translated pages look as follows::

    http://www.methods.co.nz/asciidoc/asciidoc.1.css-embedded.html

or::

    man asciidoc
