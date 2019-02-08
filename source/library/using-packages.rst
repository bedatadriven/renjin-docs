
.. _sec-using-packages:

Using Packages
--------------

When using the Renjin Script Engine, R packages are treated almost exactly
like any other Java or Scala dependency, and must be placed on your 
application's classpath by Maven or a similar build tool. 

As a service, BeDataDriven
provides a repository with all CRAN (the `Comprehensive R Archive Network`_) and
`Bioconductor`_ packages at http://packages.renjin.org. The packages in this
repository are built and packaged for use with Renjin. Not all packages can 
(yet) be built for Renjin so please consult the repository to see if your
favorite package is available for Renjin.

If you use Maven you can include a package to your project by adding it as a
dependency. For example, to include the *exptest* package you add the following
to your project's ``pom.xml`` file (don't forget to add BeDataDriven's public
repository as described in the section
:ref:`sec-project-setup`):

.. code-block:: xml

    <dependency>
        <groupId>org.renjin.cran</groupId>
        <artifactId>exptest</artifactId>
        <version>1.2-b214</version>
    </dependency>

You will find this information on the package detail page as well. For this
example this page is at http://packages.renjin.org/packages/exptest.html.
Inside your R code you can now simply attach this package to the search path
using the ``library(exptest)`` statement.


.. _Comprehensive R Archive Network: http://cran.r-project.org
.. _Bioconductor: http://bioconductor.org
