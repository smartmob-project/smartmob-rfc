RFC 2 -- .env file
==================

Abstract
~~~~~~~~

This document specifies syntax of the ``.env`` file, a format for describing
a set of environment variables that should be injected into one or more child
processes when running them.

There are several existing implementations of programs that use the ``.env``
file format and there no formal specification to be found on the Internet.
This document is an attempt to capture the syntax after the fact.

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

When running an application that follows the `The Twelve-Factor App`_
guidelines on configuration, it can be cumbersome to explicitly assign all
configuration options inside the calling environment.  In addition, it may be
impractical to do so, for example if the calling environment and application
share a common configuration option but should use different settings.

One solution for this is to run the application via a set of scripts which
assign into the environment and then invoke the application, but this makes it
impossible to use the same ``Procfile`` for both development and production,
for example.

A more popular approach is to have an optional ``.env`` file in the same
location as the ``Procfile`` which contains a set of environment variables that
should be injected into the application.

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

Each variable is declared using the following pattern::

   VAR=VAL

Whitespace on each side of the "=" character SHALL be ignored.  Trailing
whitespace SHALL be ignored.

A backslash ("\\") as the last character of a variable declaration line
indicates that the value continues on the following line.  Implementations
MUST join the lines without the line endings and treat the backslash character
as a single space.

The file MUST NOT contain any lines that are not a variable assignment, not a
comment and not blank.  Implementations MAY reject files containing non-blank,
non-comment lines that do not match a variable declaration.

.. _UTF-8: https://www.ietf.org/rfc/rfc2279.txt

Examples
~~~~~~~~

Basics
------

Example that sets a pair of variables::

  REDIS_URL=redis+tcp://localhost:6379/0
  MYSQL_URL=mysql+tcp://root:thepassword@localhost:3306/MySchema

An implementation reading this file MUST inject the following environment into
the spawned process' environment:

+---------------+----------------------------------------------------------+
| Variable      | Value                                                    |
+===============+==========================================================+
| ``REDIS_URL`` | ``redis+tcp://localhost:6379/0``                         |
+---------------+----------------------------------------------------------+
| ``MYSQL_URL`` | ``mysql+tcp://root:thepassword@localhost:3306/MySchema`` |
+---------------+----------------------------------------------------------+

Reference implementations
~~~~~~~~~~~~~~~~~~~~~~~~~

The original implementation seems to be from Foreman_, a popular utility for
running programs using a ``Procfile``.  `Foreman's man page`_ is quite brief on
the topic, simply stating that each line is a key/value pair separated by "=".

Honcho_ is a Foreman clone in Python.  Honcho's documentation on "Environment
files" provides an example but does not provide any details on the file format.

.. _Foreman: https://github.com/ddollar/foreman
.. _`Foreman's man page`: http://ddollar.github.io/foreman/
.. _Honcho: https://honcho.readthedocs.org/

Differences with existing implementations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This specification explicitly documents some hidden features supported
by other implementations:

- blank lines;
- comments;
- multiline values.

This specification explicitly requires that files be UTF-8 encoded.

Security considerations
~~~~~~~~~~~~~~~~~~~~~~~

It is commonplace to store sensitive information, such as credentials to
third-party services or attached resources in ``.env`` files.  Care should be
taken when handling these values.  For example, you may wish to place a rule on
your source control repositories to not accidentally commit the credentials.
