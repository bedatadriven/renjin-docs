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
as *GNU R*, is the reference implementation for the R language and Renjin's
current code base is derived from GNU R version 2.14.2.

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

Prerequisites
-------------

For the examples in this guide you will generally only need a Java SE
Development Kit (JDK). We recommend that you install Oracle's JDK version 6 or
7. To create Renjin extensions, as described in the chapter
:doc:`writing-renjin-extensions`, you will also need at least version 3 of Maven.

.. _sec-setting-up-a-java-project-for-renjin:

Setting up a Java project for Renjin
------------------------------------

To use Renjin in a Java application you will need to download the *Renjin
Script Engine* and its dependencies. For your convenience these can be
downloaded as a single JAR file called::

    renjin-script-engine-0.7.0-RC7-jar-with-dependencies.jar
    
from the `Renjin Script Engine module`_ in Renjin's build server.

.. _Renjin Script Engine module: http://build.bedatadriven.com/job/renjin/lastSuccessfulBuild/org.renjin$renjin-script-engine/

.. rubric:: Eclipse/IntelliJ projects

If you just want to get something up and running quickly in an IDE, you can download
the Renjin Script Engine JAR file and place it into your project's lib folder.

.. rubric:: Maven projects

For projects organized with Apache Maven, you can simply add Renjin's Script Engine as dependency to
your project:

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
        <url>http://nexus.bedatadriven.com/content/groups/public/</url>
      </repository>
    </repositories>

Evaluating R language code
--------------------------

The best way to call R from Java is to use the javax.scripting_ interfaces.
These interfaces are mature and guaranteed to be stable regardless of how
Renjin's internals evolve.

.. _javax.scripting: http://docs.oracle.com/javase/6/docs/technotes/guides/scripting/programmer_guide/

You can create a new instance of a Renjin ScriptEngine using the
ScriptEngineManager class and then instantiate Renjin's ScriptEngine using the
manager's ``getEngineByName()`` method. 

.. note::

    Unfortunately, ``ScriptEngineManager.getEngineByName()`` silently returns null
    if there are any exceptions encountered during the instantiation of Renjin's
    ScriptEngine, so you will want to check the return result and throw your own,
    more informative, exception should the creation fail.

The following code provides a template for a simple Java application that can
be used for all the examples in this guide.

.. code-block:: java

    import javax.script.*;
    // ... add additional imports here ...

    public class TryRenjin {
      public static void main(String[] args) throws Exception {
        // create a script engine manager:
        ScriptEngineManager manager = new ScriptEngineManager();
        // create a Renjin engine:
        ScriptEngine engine = manager.getEngineByName("Renjin");
        // check if the engine has loaded correctly:
        if(engine == null) {
            throw new RuntimeException("Renjin Script Engine not found on the classpath.");
        }

        // ... put your Java code here ...
      }
    }

With the ScriptEngine instance in hand, you can now evaluate R language source
code, either from a String, or from a Reader interface. The following snippet, for example,
constructs a data frame, prints it out, and then does a linear regression on the two values.

.. code-block:: java

    engine.eval("df <- data.frame(x=1:10, y=(1:10)+rnorm(n=10))");
    engine.eval("print(df)");
    engine.eval("print(lm(y ~ x, df))");

You should get output similar to the following::

       x      y     
     1  1     -0.188
     2  2      3.144
     3  3      1.625
     4  4      3.426
     5  5       6.45
     6  6       5.85
     7  7      7.774
     8  8      8.495
     9  9      9.276
    10 10     10.603

    Call:
    lm(formula = y ~ x, data = df)

    Coefficients:
    (Intercept) x          
    -0.582       1.132     

.. index::
    pair: R function; print()

.. note::

    The ScriptEngine won't print everything to standard out like the
    interactive REPL does, so if you want to output something, you'll need to
    call the R ``print()`` command explicitly.

You can also collect the R commands in a separate file

.. code-block:: r

    # script.R
    df <- data.frame(x=1:10, y=(1:10)+rnorm(n=10))
    print(df)
    print(lm(y ~ x, df))

and evaluate the script using the following snippet:

.. code-block:: java

    engine.eval(new java.io.FileReader("script.R"));

Using CRAN packages in Renjin
-----------------------------

GNU R packages can't be used directly in Renjin. As a service, BeDataDriven
provides a repository with all CRAN (the `Comprehensive R Archive Network`_)
packages at http://packages.renjin.org. The packages in this repository are
built and packaged for use with Renjin. Not all packages can be built for
Renjin so please consult the repository to see if your favorite package is
available for Renjin.

If you use Maven you can include a package to your project by adding it as a
dependency. For example, to include the *exptest* package you add the following
to your project's ``pom.xml`` file (don't forget to add BeDataDriven's public
repository as described in the section
:ref:`sec-setting-up-a-java-project-for-renjin`):

.. code-block:: xml

    <dependency>
        <groupId>org.renjin.cran</groupId>
        <artifactId>exptest</artifactId>
        <version>1.0</version>
    </dependency>

You will find this information on the package detail page as well. For this
example this page is at http://packages.renjin.org/packages/exptest.html.
Inside your R code you can now simply attach this package to the search path
using the ``library(exptest)`` statement.

.. _Comprehensive R Archive Network: http://cran.r-project.org
.. vim: tw=80
