.. The default language for highlighting source code is none:
.. highlight:: none

Writing Renjin Extensions
=========================

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
------------------------

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
------------------------------------------------

Renjin includes a built-in package called *hamcrest* for writing unit tests
using the R language. The package and its test functions are inspired by the
Hamcrest framework. From `hamcrest.org`_: *Hamcrest is a framework for writing
matcher objects allowing 'match' rules to be defined declaratively.* The
`Wikipedia article on Hamcrest`_ gives a good and short explanation of
the rationale behind the framework.

If you are familiar with the 'expectation' functions used in the testthat_
package for GNU R, then you will find many similarities with the assertion and
matcher functions in Renjin's *hamcrest* package. 

A test is a single R function with no arguments and a name that starts with
``test.``. Each test function can contain one or more assertions and the test
fails if at least one of the assertions throws an error. For example:

.. code-block:: r

    test.df <- function() {
        df <- data.frame(x = seq(10), y = runif(10))

        assertThat(df, instanceOf("data.frame"))
        assertThat(dim(df), equalTo(c(10,2)))
    }

Test functions are stored in R script files (i.e. files with extension ``.R``) in
the ``src/test/R`` folder of your package. Each file should start with the
statement ``library(hamcrest)`` in order to attach the *hamcrest* package to the
search path. You can put test functions in different files to group them
according to your liking.

The central function is the ``assertThat(actual, expected)`` function which takes
two arguments: ``actual`` is the object about which you want to make an assertion
and ``expected`` is the matcher function that defines the rule of the assertion.
In the example above, we make two assertions about the data frame ``df``, namely
that it should have class *data.frame* and that its dimension is equal to the
vector ``c(10, 2)`` (i.e. ten rows and two columns). The following sections
describe the available matcher functions in more detail.

Testing for (near) equality
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use ``equalTo()`` to test if ``actual`` is equal to ``expected``:

.. code-block:: r

    assertThat(actual, equalTo(expected))

Two objects are considered to be equal if they have the same length and if
``actual == expected`` is ``TRUE``.

Use ``identicalTo()`` to test if ``actual`` is identical to ``expected``:

.. code-block:: r

    assertThat(actual, identicalTo(expected))

Two objects are considered to be identical if ``identical(actual, expected)`` is
``TRUE``. This test is much stricter than ``equalTo()`` as it also checks that the
type of the objects and their attributes are the same. 

Use ``closeTo()`` to test for near equality (i.e. with some margin of error as
defined by the ``delta`` argument):

.. code-block:: r

    assertThat(actual, closeTo(expected, delta))

This assertion only accepts numeric vectors as arguments and ``delta`` must
have length 1. The assertion also throws an error if ``actual`` and
``expected`` do not have the same length. If their lengths are greater than 1,
the largest (absolute) difference between their elements may not exceed
``delta``.

Testing for TRUE or FALSE
~~~~~~~~~~~~~~~~~~~~~~~~~

Use ``isTrue()`` and ``isFalse()`` to check that an object is identical to ``TRUE``
or ``FALSE`` respectively:

.. code-block:: r

    assertThat(actual, isTrue())
    assertTrue(actual) # same, but shorter
    assertThat(actual, identicalTo(TRUE)) # same, but longer

Testing for class inheritance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use ``instanceOf()`` to check if an object *inherits* from a class:

.. code-block:: r

    assertThat(actual, instanceOf(expected))

An object is assumed to inherit from a class if ``inherits(actual, expected)`` is
``TRUE``.

.. tip::

    Renjin's *hamcrest* package also exists as a GNU R package with the same
    name available at https://github.com/bedatadriven/hamcrest. If you are
    writing a package for both Renjin and GNU R, you can use the *hamcrest*
    package to check the compatibility of your code by running the test files
    in both Renjin and GNU R.

.. _Writing R Extensions: http://cran.r-project.org/doc/manuals/r-release/R-exts.html
.. _Maven standard directory layout: http://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html
.. _DESCRIPTION: http://cran.r-project.org/doc/manuals/r-release/R-exts.html#The-DESCRIPTION-file
.. _NAMESPACE: http://cran.r-project.org/doc/manuals/r-release/R-exts.html#Package-namespaces
.. _hamcrest.org: https://code.google.com/p/hamcrest/wiki/Tutorial
.. _Wikipedia article on Hamcrest: http://en.wikipedia.org/wiki/Hamcrest
.. _testthat: http://cran.r-project.org/web/packages/testthat/index.html
