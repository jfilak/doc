.. _resolv:

Debugging crashes reported by abrt
==================================

Signals
-------

* Signal 5  (`SIGTRAP`) = "General crash"
* Signal 6  (`SIGABRT`) = SIGABRT is commonly used by ``libc`` and other libraries to abort the 
  program in case of critical errors. For example, ``glibc`` sends an SIGABRT in case of a detected double-free or other heap corruptions.
* Signal 11 (`SIGSEGV`) = Segmentation fault, bus error, or access violation. It is generally an attempt to access memory that the CPU cannot physically address.
