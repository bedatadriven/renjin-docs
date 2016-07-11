.. highlight:: none

Introduction
============

This guide covers Renjin version |release| and is aimed at developers looking
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


.. _Comprehensive R Archive Network: http://cran.r-project.org
.. vim: tw=80
