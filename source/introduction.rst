.. highlight:: none

Introduction
============

This guide covers Renjin version |version| and is aimed at developers looking
to:

1. integrate R code in their Java applications and to exchange data between Java
   and R code, and/or to
2. create extension packages that can be used by Renjin much like packages are
   used to extend GNU R's functionality.

The guide also covers the parts of Renjin's Java API that are most relevant to
these goals.

About Renjin
------------

Renjin is an interpreter for the R programming language for statistical
computing written in Java much like JRuby_ and Jython_ are for the Ruby and
Python programming languages. The official `R project`_, hereafter referred to
as *GNU R*, is the reference implementation for the R language.

The goal of Renjin is to eventually be compatible with GNU R such that most
existing R language programs will run in Renjin without the need to make any
changes to the code. Needless to say, Renjin is currently not 100% compatible
with GNU R so your mileage may vary.

Executing R code from Java or visa versa is not new: there is the rJava_ package
for GNU R that allows R code to call Java methods and there is the RCaller_
package to call R from Java which is similar to the JRI_ package (now shipped as
part of *rJava*). *JRI* loads the R dynamic library into Java and provides a
Java API to the R functionality. *RCaller* works a little different by running R
as a separate process that the package communicates with. Finally, *rJava* uses
the Java Native Interface (JNI) to enable R to call Java applications running in
the JVM.

The biggest advantage of Renjin is that the R interpreter itself is a Java
module which can be seamlessly integrated into any Java application. This
dispenses with the need to load dynamic libraries or to provide some form of
communication between separate processes. These types of interfaces are often
the `source of much agony`_ because they place very specific demands on the
environment in which they run. 

Renjin also benefits from the Java ecosystem which, amongst many things,
includes professional tools for component (or application) life-cycle
management. Think of `Apache Maven`_ as a software project management tool for
building components (i.e. artifacts in Maven parlance) and managing dependencies
as well as the likes of Artifactory_ and Nexus_ for repository management.

Another advantage of Renjin is that no extra sauce is required to enable the
R/Java interface. Packages like *rJava* and *RCaller* require that you litter
your code with special commands that are part of their API. As the chapter
:doc:`importing-java-classes-in-r-code` shows, Renjin provides direct access to
Java methods using an interface that is completely unobtrusive.

See http://www.renjin.org for more information on Renjin.

.. _JRuby: http://www.jruby.org
.. _Jython: http://www.jython.org
.. _R project: http://www.r-project.org
.. _rJava: http://www.rforge.net/rJava/
.. _RCaller: https://code.google.com/p/rcaller/
.. _JRI: http://www.rforge.net/JRI
.. _source of much agony: http://stackoverflow.com/tags/rjava/hot
.. _Apache Maven: http://maven.apache.org
.. _Artifactory: http://www.jfrog.com
.. _Nexus: http://www.sonatype.org/nexus/


Understanding Renjin and package versions
-----------------------------------------

We version two things: Renjin itself and the individual extension packages which
we build for Renjin.

.. index::
   single: version

Versions and builds of Renjin
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Renjin version number consists of two pieces of information: the major
version number and the build number:

.. _fig-renjin-version:

.. figure:: /images/renjin-version.png

    Renjin version numbering

Every time we commit a change to `Renjin's source on GitHub`_, a build job is
automatically triggered on our build server which assigns the build number to
the Renjin version number. If the build succeeds, the artifacts are deployed to
our public repository.

The build number in Renjin's version number always increases and is independent
of the major version (i.e. it isn't reset to 1 when we increase the major
version).

.. _Renjin's source on GitHub: https://github.com/bedatadriven/renjin 

Package versions and builds
~~~~~~~~~~~~~~~~~~~~~~~~~~~

R extension packages from CRAN and Bioconductor have their own version numbers
which we also use in Renjin. Depending on what changes were committed to
Renjin's source, we will manually trigger a build of packages, either all 10000+
of them or a random selection, to assess the effect of the changes on the test
results.

Following the explanation in `this blog post`_, to fully reference packages in
Renjin one would use the following format:

.. _fig-package-version:

.. figure:: /images/package-version.png

    Version numbering of Renjin-compatible extension packages

The labels at the top correspond to the fields in a Maven project (POM) file
whereas the bottom labels explain how package references are constructed. The
package detail page in Renjin's package repository browser tells you how to load
extension packages from the command line or using a POM file (see the section
:ref:`sec-using-r-packages-in-renjin`).

.. _this blog post: http://www.renjin.org/blog/2015-09-14-new-packages-renjin-org.html

.. vim: tw=80

