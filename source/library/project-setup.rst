
.. _sec-project-setup:

Project Setup
-------------

Renjin is a essentially a Java library that allows you to evaluate scripts
written in the R language. This library must be added as dependency to
your project.

.. contents::  :local:
  
Maven
~~~~~

For projects organized with Apache Maven, you can simply add 
Renjin's Script Engine as dependency to your project:

.. code-block:: xml

   <dependencies>
     <dependency>
       <groupId>org.renjin</groupId>
       <artifactId>renjin-script-engine</artifactId>
       <version>0.8.2443</version>
     </dependency>
   </dependencies>

For this to work you will also need to add BeDataDriven's public repository to your ``pom.xml``:

.. code-block:: xml

    <repositories>
      <repository>
        <id>bedatadriven</id>
        <name>bedatadriven public repo</name>
        <url>https://nexus.bedatadriven.com/content/groups/public/</url>
      </repository>
    </repositories>

You can use ``RELEASE`` instead of ``0.8.2443`` in the project file to use the
very latest versions of the Renjin components.


Gradle
~~~~~~

For projects organized with Gradle, add the following to your ``build.gradle`` file:

.. code-block:: groovy
    
    repositories {
      maven { url "https://nexus.bedatadriven.com/content/groups/public" }
    }
    
    dependencies {
      compile "org.renjin:renjin-script-engine:0.8.2443";
    }

See the `renjin-gradle-example`_ on GitHub for a complete example.

.. _renjin-gradle-example: https://github.com/bedatadriven/renjin-gradle-example



Scala Build Tool (SBT)
~~~~~~~~~~~~~~~~~~~~~~

The following is an example of ``build.sbt`` that includes 
Renjin's Script Engine:

.. code-block:: scala
 
    // IMPORTANT: sbt may fail if http*s* is not used.
    resolvers += 
        "BeDataDriven" at "https://nexus.bedatadriven.com/content/groups/public"

    lazy val root = (project in file(".")).
      settings(
        name := "renjin-test",
        version := "1.0",
        scalaVersion := "2.10.6",
        libraryDependencies += "org.renjin" % "renjin-script-engine" % "0.8.2443"
      )

See the `renjin-sbt-example`_ on GitHub for a complete example.

.. note::

    There has been a `report`_ that the `coursier`_ plugin fails to resolve
    Renjin's dependencies. If you encounter class path problems with the plugin,
    try building your project without.

.. _renjin-sbt-example: https://github.com/bedatadriven/renjin-gradle-example
.. _report: http://stackoverflow.com/questions/40888063/load-rdata-from-an-r-script-in-scala-using-renjin#answer-40999169
.. _coursier: https://github.com/alexarchambault/coursier

      
Eclipse
~~~~~~~

We recommend using a build tool to organize your project. As soon as you begin 
using non-trivial R packages, it will become increasingly difficult to
manage dependencies (and the dependencies of those dependencies) 
through a point-and-click interface. 

If this isn't possible for whatever reason, you can download 
a single JAR file called:

    renjin-script-engine-0.8.2443-jar-with-dependencies.jar

from the Renjin website and manually add this as a dependency in Eclipse.

See the `eclipse-dynamic-web-project`_ example project for more details.

.. _eclipse-dynamic-web-project: https://github.com/bedatadriven/renjin-examples/tree/master/eclipse-dynamic-web-project

JBoss
~~~~~

There have been reports of difficulty loading Renjin within JBoss without
a specific ``module.xml`` file:

.. code-block:: xml

    <module xmlns="urn:jboss:module:1.1" name="org.renjin">
      <resources>
        <resource-root path="renjin-script-engine-0.8.2443-jar-with-dependencies.jar"/>
      </resources>
      <dependencies>
        <module name="javax.api"/>
      </dependencies>
    </module>


Spark
~~~~~

The `spark-submit` command line tool requires you to explicitly specify the dependencies
of your Spark Job. In order to avoid specifying all of Renjin's dependencies,
as well as those of CRAN, and BioConductor packages, or your own internal
packages, you can still use Maven (or Gradle or SBT) to automatically resolve 
your dependencies and build a single JAR that you can pass as an argument 
to `spark-submit` or `dse spark-submit`.

.. code-block:: xml

    <dependencies>
      <dependency>
        <groupId>com.datastax.dse</groupId>
        <artifactId>dse-spark-dependencies</artifactId>
        <version>5.0.1</version>
        <scope>provided</scope>
      </dependency>
      
      <dependency>
        <groupId>org.renjin</groupId>
        <artifactId>renjin-script-engine</artifactId>
        <version>0.8.2443</version>
      </dependency>
   
      <dependency>
        <groupId>org.renjin.cran</groupId>
        <artifactId>randomForest</artifactId>
        <version>4.6-12-b34</version>
      </dependency>
    </dependencies>

    <build>
      <!--- Assembly plugin to build single jar -->
    </build>
    
    <repositories>
      <!-- Renjin and Spark/DataStax repositories -->
    </repositories>
     
See the `renjin-spark-executor`_ project or the 
`datastax/SparkBuildExamples`_ repository from DataStax for complete examples.

.. _renjin-spark-executor: https://github.com/onetapbeyond/renjin-spark-executor/tree/master/examples/java/hello-world
.. _datastax/SparkBuildExamples: https://github.com/datastax/SparkBuildExamples

You can then submit your job as follows:

.. code-block:: sh

   mvn clean package
   spark-submit --class org.renjin.ExampleJob target/renjin-example-0.1-dep.jar




