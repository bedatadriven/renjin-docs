.. highlight:: none

Introduction
============

This guide intended for Java developers who wish to call R code from a Java
application, webserver, or other project. This guide is for Renjin version
|release|. In the following examples, replace ``RENJIN_VERSION`` with
|release|.

.. The approach is generally the same for other JVM languages such as Scala,
.. Clojure, JRuby, etc, but users of those languages will need to make some mental
.. translation from Java syntax to their own.

Before we start, it is good to realize that in these examples we are not
*calling R* as in GNU R. Technically, we have statements written in the
syntax of the R programming language and we are executing these statements using
the interpreter provided by Renjin. This is what makes Renjin different
from packages like rJava_ and rcaller_. 

.. _rJava: http://www.rforge.net/rJava/
.. _rcaller: https://code.google.com/p/rcaller/

.. _sec-setting-up-a-java-project-for-renjin:

Setting up a Java project for Renjin
------------------------------------

To use Renjin in a Java application you will need to download the *Renjin
Script Engine* and its dependencies. For your convenience these can be
downloaded as a single JAR file called::

    renjin-script-engine-RENJIN_VERSION-jar-with-dependencies.jar
    
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
       <version>RENJIN_VERSION</version>
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

    engine.eval(new java.io.FileReader("script.R"))

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
