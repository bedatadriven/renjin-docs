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
your R code. You can use Javadoc to document your Java classes and methods.

Package directory layout
------------------------

The files in a Renjin package must be organized in a directory structure that
adheres to the `Maven standard directory layout`_. A directory layout that will
cover most Renjin packages is as follows::

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
    NAMESPACE               Almost equivalent to R's NAMESPACE file
    pom.xml                 Maven's Project Object Model file
    ====================    =======================

The functionality of the *DESCRIPTION* file used by GNU R packages is replaced
by a Maven *pom.xml* (POM) file. In this file you define the name of your
package and any dependencies, if applicable. The POM file is used by Renjin's
Maven plugin to create the package. This is the subject of the next section.

Renjin Maven plugin
-------------------

Whereas you would use the commands ``R CMD check``, ``R CMD build``, and ``R
CMD INSTALL`` to check, build (i.e. package), and install packages for GNU R,
packages for Renjin are tested, packaged, and installed using a Maven plugin.
The following XML file can be used as a *pom.xml* template for all Renjin
packages:

.. code-block:: xml

    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
             http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>

        <groupId>com.acme</groupId>
        <artifactId>foobar</artifactId>
        <version>1.0-SNAPSHOT</version>
        <packaging>jar</packaging>

        <!-- general information about your package -->
        <name>Package name or title</name>
        <description>A short description of your package.</description>
        <url>http://www.url.to/your/package/website</url>
        <licenses>
            <!-- add one or more licenses under which the package is released -->
            <license>
                <name>Apache License version 2.0</name>
                <url>http://www.apache.org/licenses/LICENSE-2.0.html</url>
            </license>
        </licenses>

        <properties>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            <renjin.version>0.8.2327</renjin.version>
        </properties>

        <dependencies>
            <!-- the script engine is convenient even if you do not use it explicitly -->
            <dependency>
                <groupId>org.renjin</groupId>
                <artifactId>renjin-script-engine</artifactId>
                <version>${renjin.version}</version>
            </dependency>
            <!-- the hamcrest package is only required if you use it for unit tests -->
            <dependency>
                <groupId>org.renjin</groupId>
                <artifactId>hamcrest</artifactId>
                <version>${renjin.version}</version>
                <scope>test</scope>
            </dependency>
        </dependencies>

        <repositories>
            <repository>
                <id>bedatadriven</id>
                <name>bedatadriven public repo</name>
                <url>https://nexus.bedatadriven.com/content/groups/public/</url>
            </repository>
        </repositories>

        <pluginRepositories>
            <pluginRepository>
                <id>bedatadriven</id>
                <name>bedatadriven public repo</name>
                <url>https://nexus.bedatadriven.com/content/groups/public/</url>
            </pluginRepository>
        </pluginRepositories>

        <build>
            <plugins>
                <plugin>
                    <groupId>org.renjin</groupId>
                    <artifactId>renjin-maven-plugin</artifactId>
                    <version>${renjin.version}</version>
                    <executions>
                        <execution>
                            <id>build</id>
                            <goals>
                                <goal>namespace-compile</goal>
                            </goals>
                            <phase>process-classes</phase>
                        </execution>
                        <execution>
                            <id>test</id>
                            <goals>
                                <goal>test</goal>
                            </goals>
                            <phase>test</phase>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </project>

This POM file provides a lot of information:

* fully qualified name of the package, namely *com.acme.foobar*;
* package version, namely *1.0-SNAPSHOT*;
* package dependencies and their versions, namely the Renjin Script Engine and
  the *hamcrest* package (see the next section);
* BeDataDriven's public repository to look for the dependencies if it can't
  find them locally or in Maven Central;

.. important::

    Package names is one area where Renjin takes a different approach to GNU R
    and adheres to the Java standard of using fully qualified names. The
    package in the example above must be loaded using its fully qualified name,
    that is with ``library(com.acme.foobar)`` or ``require(com.acme.foobar)``.
    The group ID (*com.acme* in this example) is traditionally a domain over
    which only you have control. The artifact ID should have only `lower case
    letters and no strange symbols`_. The term *artifact* is used by Maven to
    refer to the result of a build which, in the context of this chapter, is
    always a package.

Now you can use Maven to test, package, and install your package using the
following commands:

mvn test
    run the package tests (both the Java and R code tests)

mvn package
    create a JAR file of the package (named *foobar-1.0-SNAPSHOT.jar* in the
    example above) in the ``target`` folder of the package's root directory

mvn install
    install the artifact (i.e. package) into the local repository

mvn deploy
    upload the artifact to a remote repository (requires additional
    configuration)

mvn clean
    clean the project's working directory after a build (can also be combined
    with one of the previous commands, for example: ``mvn clean install``)


Package NAMESPACE file
----------------------

Since R version 2.14, packages are required to have a ``NAMESPACE`` file and the
same holds for Renjin. Because of dynamic searching for objects in R, the use of
a ``NAMESPACE`` file is good practice anyway. The ``NAMESPACE`` file is used to
explicitly define which functions should be *imported* into the package's
namespace and which functions the package exposes (i.e. *exports*) to other
packages. Using this file, the package developer controls how his or her package
finds functions.

Usage of the ``NAMESPACE`` in Renjin is almost exactly the same as in GNU R
save for two differences:

1. the directives related to S4 classes are not yet supported by Renjin and
2. Renjin accepts the directive ``importClass()`` for importing Java classes
   into the package namespace. 

Here is an overview of the namespace directives that Renjin supports:

``export(f)`` or ``export(f, g)``
    Export an object ``f`` (singular form) or multiple objects ``f`` and ``g``
    (plural form). You can add as many objects to this directive as you like.

``exportPattern("^[^\\.]")``
    Export all objects whose name does not start with a period ('.').
    Although any regular expression can be used in this directive, this is by
    far the most common one. It is considered to be good practice not to use
    this directive and to explicitly export objects using the ``export()``
    directive.

``import(foo)`` or ``import(foo, bar)``
    Import all exported objects from the package named ``foo`` (and ``bar``
    in the plural form). Like the ``export()`` directive, you can add as many
    objects as you like to this directive.

``importFrom(foo, f)`` or ``importFrom(foo, f, g)``
    Import only object ``f`` (and ``g`` in the plural form) from the package
    named ``foo``.
    
``S3method(print, foo)``
    Register a print (S3) method for the ``foo`` class. This ensures that other
    packages understand that you provide a function ``print.foo()`` that is a
    print method for class ``foo``. The ``print.foo()`` does not need to be
    exported.

``importClass(com.acme.myclass)``
    A namespace directive which is unique to Renjin and which allows Java
    classes to be imported into the package namespace. This directive is
    actually a function which does the same as Renjin's ``import()`` function
    that was introduced in the chapter :doc:`importing-java-classes-in-r-code`.

To summarize: the R functions in your package have access to all R functions
defined within your package (also those that are not explicitely exported) as
well as the Java classes imported into the package names using the
``importClass`` directive. Other packages only have access to the R objects
that your package exports as well as to the public Java classes. Since Java has
its own mechanism to control the visibility of classes, there is no
``exportClass`` directive in the ``NAMESPACE`` file.s

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
fails if at least one of the assertions throws an error. For example, using the
package defined in the previous section:

.. code-block:: r

    library(hamcrest)
    library(com.acme.foobar)

    test.df <- function() {
        df <- data.frame(x = seq(10), y = runif(10))

        assertThat(df, instanceOf("data.frame"))
        assertThat(dim(df), equalTo(c(10,2)))
    }

Test functions are stored in R script files (i.e. files with extension ``.R``
or ``.S``) in the ``src/test/R`` folder of your package. Each file should start
with the statement ``library(hamcrest)`` in order to attach the *hamcrest*
package to the search path as well as a ``library()`` statement to load your
own package.  You can put test functions in different files to group them
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

Understanding test results
~~~~~~~~~~~~~~~~~~~~~~~~~~

When you run ``mvn test`` within the directory that holds the POM file (i.e.
the root directory of your package), Maven will execute both the Java and R
unit tests and output various bits of information including the test results.
The results for the Java tests are summarized in a section marked with::

    -------------------------------------------------------
     T E S T S
    -------------------------------------------------------

and which will summarize the test results like::

    Results :

    Tests run: 5, Failures: 1, Errors: 0, Skipped: 0

The results of the R tests are summarized in a section marked with::

    -------------------------------------------------------
     R E N J I N   T E S T S
    -------------------------------------------------------

The R tests are summarized per R source file which will look similar to the
following example::

    Running tests in /home/foobar/mypkg/src/test/R
    Running function_test.R
    No default packages specified
    Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.898 

Note that the number of tests run is equal to the number of ``test.*``
functions in the R source file + 1 as running the test file is also counted as
a test. 

.. _lower case letters and no strange symbols: http://maven.apache.org/guides/mini/guide-naming-conventions.html


