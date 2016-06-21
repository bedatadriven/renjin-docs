
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
       <version>0.7.0-RC7</version>
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

You can use ``RELEASE`` instead of ``0.7.0-RC7`` in the project file to use the
very latest versions of the Renjin components.


Gradle
~~~~~~

For projects organized with Gradle, add the following to your ``build.gradle`` file:

.. code-block:: groovy
    
    repositories {
      maven { url "https://nexus.bedatadriven.com/content/groups/public" }
    }
    
    dependencies {
      compile "org.renjin:renjin-script-engine:0.7.0-RC7";
    }

Scala Build Tool (SBT)
~~~~~~~~~~~~~~~~~~~~~~

The following is an example of ``build.sbt`` that includes 
Renjin's Script Engine:

.. code-block:: scala

    resolvers += 
        "BeDataDriven" at "https://nexus.bedatadriven.com/content/groups/public"

    lazy val root = (project in file(".")).
      settings(
        name := "renjin-test",
        version := "1.0",
        scalaVersion := "2.10.6",
        libraryDependencies += "org.renjin" % "renjin-script-engine" % "0.8.1886"
      )
      
Eclipse
~~~~~~~

We recommend using a build tool to organize your project. As soon as you begin 
using non-trivial R packages, it will become increasingly difficult to
manage dependencies (and the dependencies of those dependencies) 
through a point-and-click interface. 

If this isn't possible for whatever reason, you can download 
a single JAR file called:

    renjin-script-engine-0.7.0-RC7-jar-with-dependencies.jar

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
        <resource-root path="renjin-script-engine-0.8.1908-jar-with-dependencies.jar"/>
      </resources>
      <dependencies>
        <module name="javax.api"/>
      </dependencies>
    </module>


