

Capturing results from Renjin
----------------------------

There are two main options for capturing results from R code 
evaluated by the Renjin ScriptEngine. You can either capture the 
*printed* output of a function, *or* access individual values.

Let's take the example of fitting an SVM model with the `e1071 package`_.


.. code-block:: java

    import javax.script.*;
    import org.renjin.script.*;

    public class SVM {
      public static void main(String[] args) throws Exception {
        // create a script engine manager:
        RenjinScriptEngineFactory factory = new RenjinScriptEngineFactory();
        // create a Renjin engine:
        ScriptEngine engine = factory.getScriptEngine();
	engine.eval("data(iris)");
        engine.eval("svmfit <- svm(Species~., data=iris)");
      }
    }

Right now, the fitted model is stored in the variable *svmfit* inside the 
R session, but is not yet accessible to the Java program.

Capturing output text
~~~~~~~~~~~~~~~~~~~~~

The simplest thing we can do is to ask the svm package to print a summary
of the model to the standard output stream:

.. code-block:: java

   engine.eval("print(svmfit)");


However, by default, Renjin's standard output stream will just print to your 
console, yielding::

    Call:
    svm(data = iris, formula = Species ~ .)
    
    Parameters:
    SVM-Type:  C-classification 
    SVM-Kernel:  radial 
          cost:  1 
         gamma:  0.25 
    
    Number of Support Vectors:  45

This is helpful, but the text is not yet accessible to our Java program.
To store this text to a Java string, we can redirect Renjin's output to 
a ``StringWriter``.

.. code-block:: java

    StringWriter outputWriter = new StringWriter();
    engine.getContext().setWriter(outputWriter);
    engine.eval("print(svmfit)");
    
    String output = outputWriter.toString();

    // Reset output to console
    engine.getContext().setWriter(new PrintWriter(System.out));

Now the output of the ``print()`` function call is stored in the Java
`output` variable.


Extracting individual values
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You will most likely want to access individual values rather than simply
output text.

The `svmfit` variable in the R session, however, holds a complicated R object,
however, built with lists of lists. 

You can look at the structure of this object using the str() function::

    > str(svmfit)
    List of 30
     $ call           :length 3 svm(data = iris, formula = Species ~ .)
     $ type           : num 0
     $ kernel         : num 2
     $ cost           : num 1
     $ degree         : num 3
     $ gamma          : num 0.25
     $ coef0          : num 0
     $ nu             : num 0.5
     $ epsilon        : num 0.1
     $ sparse         : logi FALSE
     $ scaled         : logi [1:4] FALSE FALSE FALSE FALSE
     $ x.scale        : NULL
     $ y.scale        : NULL
     $ nclasses       : int 3
     $ levels         : chr [1:3] "setosa" "versicolor" "virginica"
     $ tot.nSV        : int 45
     $ nSV            : int [1:3] 7 19 19
     $ labels         : int [1:3] 1 2 3

     ... etc ...

Now we can see that svmfit object is an R list with 30 named properties,
including "cost", "type", "gamma", etc.

We can ask the Renjin ScriptEngine for these values and then use the results
in our Java program. For example:

.. code-block:: java

    Vector gammaVector = (Vector)engine.eval("svmfit$gamma");
    double gamma = gammaVector.getElementAsDouble(0);

    Vector nclassesVector = (Vector)engine.eval("svmfit$nclasses");
    int nclasses = nclasses = nclassesVector.getElementAsInt(0);

    StringVector levelsVector = (StringVector)engine.eval("svmfit$levels");
    String[] levelsArray = levelsVector.toArray();

The ``engine.eval()`` method will always return an object of type ``SEXP``,
which is the Java type Renjin uses to represent R's "S-Expressions". You can 
read more about these types and how to access their values in the `javadoc`_.

.. _e1071 package: http://packages.renjin.org/package/org.renjin.cran/e1071
.. _javadoc: http://javadoc.renjin.org/latest/index.html?org/renjin/sexp/package-summary.html





