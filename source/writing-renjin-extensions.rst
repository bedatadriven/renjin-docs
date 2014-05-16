.. The default language for highlighting source code is none:
.. highlight:: none

Writing Renjin Extensions
-------------------------

This chapter can be considered as Renjin's equivalent of the `Writing R
Extensions`_ manual for GNU R. Here we discuss how you create extensions, or
*packages* as they are referred to in R, for Renjin. Packages allow you to
group logically related functions and data together in a single archive which
can be reused and shared with others.

Renjin packages do not differ much from packages for GNU R. One notable
difference is that Renjin treats unit tests as first class citizens, i.e.
Renjin includes a package called *hamcrest* that provides functionality for
writing unit tests right out of the box. We encourage you to include as many
unit tests with your package as possible.

One feature currently missing for Renjin packages is the ability to document
your code. Sorry.

Package directory layout
~~~~~~~~~~~~~~~~~~~~~~~~

Like R packages, the files in a Renjin package must be organized in a directory
structure that adheres to the `Maven standard directory layout`_. A directory
layout that will cover most Renjin packages is as follows::

    projectdir/
        src/
            main/
                java/
                    ...
                R/
                resources/
            test/
                java/
                    ...
                R/
                resources/
        DESCRIPTION
        NAMESPACE
        pom.xml

The table :ref:`tab-renjin-package-directory-layout` gives a short description
of the directories and files in this layout.

.. _tab-renjin-package-directory-layout:

.. table:: Directories in a Renjin package

    ====================    =======================
    Directory               Description
    ====================    =======================
    src/main/java           Java source code (\*.java files)
    src/main/R              R source code (\*.R files)
    src/main/resources      Files and directories to be copied to the root of the generated JAR file
    src/test/java           Unit tests written in Java using JUnit
    src/test/R              Unit tests written in R using Renjin's Hamcrest package
    src/test/resource       Files available to the unit tests (not copied into the generated JAR file)
    DESCRIPTION             Equivalent to R's DESCRIPTION file
    NAMESPACE               Almost equivalent to R's NAMESPACE file
    pom.xml                 Maven's Project Object Model file
    ====================    =======================

Using the *hamcrest* package to write unit tests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Docs for the hamcrest package.

.. _Writing R Extensions: http://cran.r-project.org/doc/manuals/r-release/R-exts.html
.. _Maven standard directory layout: http://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html
.. _DESCRIPTION: http://cran.r-project.org/doc/manuals/r-release/R-exts.html#The-DESCRIPTION-file
.. _NAMESPACE: http://cran.r-project.org/doc/manuals/r-release/R-exts.html#Package-namespaces
