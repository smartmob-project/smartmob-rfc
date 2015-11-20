RFC 1 -- Procfile
=================

Abstract
~~~~~~~~

This document specifies syntax of the ``Procfile``, a format for describing the
services that make up a distributed system or "app".

There are several existing implementations of programs that use the
``Procfile`` format and there no formal specification to be found on the
Internet.  This document is an attempt to capture the syntax after the fact.

Preamble
~~~~~~~~

Copyright (c) 2015 smartmob contributors

This Specification is free software; you can redistribute it and/or modify it
under the terms of the GNU General Public License as published by the Free
Software Foundation; either version 3 of the License, or (at your option) any
later version. This Specification is distributed in the hope that it will be
useful, but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public
License for more details. You should have received a copy of the GNU General
Public License along with this program; if not, see
<http://www.gnu.org/licenses>.

Problem statement
~~~~~~~~~~~~~~~~~

When sharing code for an application (or "app") that follows guidelines from
`The Twelve-Factor App`_ expects a means to specify the entry points for its
stateless processes.

The goal of this specification is to describe a clear, concise file format for
specifying these entry points.

.. _`The Twelve-Factor App`: http://12factor.net/

Formal specification
~~~~~~~~~~~~~~~~~~~~

All text MUST be UTF-8_ encoded, without BOM.  Implementations SHOULD reject
files that contain invalid UTF-8 data.

Leading whitespace is ignored for all lines, including commands, blank lines
and comments.

Any blank line (possibly containing only whitespace) MUST be ignored.

Any line starting with a hash/pound ("#") is considered a comment for human
readers and is ignored.

Each process type is declared using the following pattern::

   <process type>: <command>

A command may start with a "variable assignment" as in popular UNIX shells.
Implementations MUST strip this from the command and inject the variable into
the child process' environment variables.

A backslash ("\\") as the last character of a process type line indicates that
the command continues on the following line.  Implementations MUST join the
lines without the line endings and treat the backslash character as a single
space.

The file MUST NOT contain any lines that are not a command, not a comment and
not blank.  Implementations MAY reject files containing non-blank, non-comment
lines that do not match a process type declaration.

.. _UTF-8: https://www.ietf.org/rfc/rfc2279.txt

Examples
~~~~~~~~

Basics
------

::

   web: gunicorn myapp:app
   worker: celery -A tasks worker --loglevel=info

This ``Procfile`` defines a ``web`` process type that runs gunicorn_ using the
``gunicorn myapp:app`` command and a ``worker`` process type that runs the
celery_ worker using the ``celery -A tasks worker --loglevel=info`` command.

.. _gunicorn: http://gunicorn.org/
.. _celery: http://www.celeryproject.org/

Blank lines and comments
------------------------

::

   # OMG gunicorn is fast!
   web: gunicorn myapp:app

   # OMG celery is easy!
   worker: celery -A tasks worker --loglevel=info


Multiline commands
------------------

::

   web: gunicorn \
     myapp:app
   worker: \
     celery \
     -A tasks \
     worker \
     --loglevel=info

Reference implementations
~~~~~~~~~~~~~~~~~~~~~~~~~

The original implementation seems to be from Heroku_.  Heroku's documentation
on `Declaring process types
<https://devcenter.heroku.com/articles/procfile#declaring-process-types>`_
contains a short, informal and incomplete description of the syntax supported
by Heroku.

Foreman_ is a popular utility for running programs using a ``Procfile``.  The
project page links to other implementations.

.. _Heroku: https://www.heroku.com
.. _Foreman: https://github.com/ddollar/foreman

Differences with existing implementations
-----------------------------------------

In addition to the documented behavior, Heroku and Foreman seem to support some
additional hidden functionality such as:

- ignoring blank lines, for legibility;
- ignoring invalid lines, which can be useful for adding in-line comments;
- prefixing commands with environment variable assignments.

This specification attempts to specify these hidden features.

In addition, it adds support for multiline commands.

Security considerations
~~~~~~~~~~~~~~~~~~~~~~~

You should take care when running a program that executes commands in a
``Procfile`` that comes from an untrusted source.  Running the untrusted
commands inside a sandbox_, chroot_ jail, `virtual machine`_ or container_ may
offer some form of protection.

Applications that use a ``Procfile`` typically expect to be able to "scale" by
spawning multiple instances of some process types described in the
``Procfile``.  However, it is also common to specify the entry point for a
singleton process (for which there should be no more than 1 instance at any
given time) and to specify the entry point for administrative "one-off"
commands that should not run as part of the usual set of instances.  Running an
incorrect number of instances may cause unexpected results.  Before running
commands in a ``Procfile``, you should read the application's documentation to
know how many instances of each process type you can/should run.

.. _sandbox: https://en.wikipedia.org/wiki/Sandbox_(computer_security)
.. _chroot: https://en.wikipedia.org/wiki/Chroot
.. _container: https://en.wikipedia.org/wiki/Operating-system-level_virtualization
.. _`virtual machine`: https://en.wikipedia.org/wiki/Virtual_machine
