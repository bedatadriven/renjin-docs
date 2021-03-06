

Evaluating R Language Code
--------------------------

The best way to call R from Java is to use the javax.scripting_ interfaces.
These interfaces are mature and guaranteed to be stable regardless of how
Renjin's internals evolve.

.. _javax.scripting: http://docs.oracle.com/javase/6/docs/technotes/guides/scripting/programmer_guide/

You can create a new instance of a Renjin ScriptEngine using the
RenjinScriptEngineFactory class and then instantiate Renjin's ScriptEngine using the
factory's ``getEngine()`` method. 


The following code provides a template for a simple Java application that can
be used for all the examples in this guide.

.. code-block:: java

    import javax.script.*;
    import org.renjin.script.*;

    // ... add additional imports here ...

    public class TryRenjin {
      public static void main(String[] args) throws Exception {
        // create a script engine manager:
        RenjinScriptEngineFactory factory = new RenjinScriptEngineFactory();
        // create a Renjin engine:
        ScriptEngine engine = factory.getScriptEngine();

        // ... put your Java code here ...
      }
    }

.. note::

    We recommend using ``RenjinScriptEngineFactory`` directly, as the standard 
    javax.script silently returns null and hides any exceptions encountered when
    loading Renjin, making it very difficult to debug any project setup problems.

    If you're using Renjin in a more generic context, you can load the engine by name
    by calling ``ScriptEngineManager.getEngineByName("Renjin")``.

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

