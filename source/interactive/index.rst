
Using Renjin Interactively
==========================

.. index::
   single: REPL

Though Renjin's principle goal is to make it easier to embed R code 
in existing systems, it can also be used as an interactive 
Read-Eval-Print-Loop (REPL) similar to that of GNU R.

.. figure:: /images/terminal.png

   Interactive interpreter run from the command line
   

Installation
------------

Visit the `downloads page <http://www.renjin.org/downloads.html>`_ on renjin.org.


Using packages
--------------

There are some differences between the way Renjin manages packages compared
to the way that GNU R manages packages.

In GNU R, you must first run ``install.packages()``, which will download
and build a package from source. After the package is installed, then it can
be loaded with a call to ``library()``.

.. index::
   pair: R function; library()
   pair: R function; install.packages()

From within Renjin's REPL, there is no ``install.packages()`` function: the 
first time you try to load a package with ``library()``, Renjin will 
check the repository for a package with the matching name and download it to 
a local repository located in ``~/.m2/repository``.

As a service, BeDataDriven provides a repository with all CRAN
(the `Comprehensive R Archive Network`_) and BioConductor packages at
http://packages.renjin.org. The packages in this repository are
built and packaged for use with Renjin. Not all packages can be built for
Renjin so please consult the repository to see if your favorite package is
available for Renjin.

.. _Comprehensive R Archive Network: https://cran.r-project.org

