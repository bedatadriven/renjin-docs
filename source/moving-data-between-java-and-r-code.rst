.. The default language for highlighting source code is none:
.. highlight:: none

Moving data between Java and R code
===================================

If you read the :doc:`introduction` to this guide you already know how to
execute R code from a Java application. In this chapter we will take things a
little further and explain how you can move data between Java and R code.

Renjin provides a mapping from R language types to Java objects. To use this
mapping effectively you should have at least a basic understanding of R's object
types.  The next section provides a short introduction which is essentially a
condensed version of the relevant material in the `R Language Definition
manual`_. If you are already familiar with R's object types you can skip this
section and head straight to the section :ref:`sec-pulling-data-from-r-into-java` or
:ref:`sec-pushing-data-from-java-to-r`.

.. _R Language Definition manual: http://cran.r-project.org/doc/manuals/r-release/R-lang.html

.. _sec-java-developer-guide-to-r-objects:

A Java developer's guide to R objects
-------------------------------------

R has a number of objects types that are referred to as *basic types*. Of these,
we only discuss those that are most frequently encountered by users of R:
vectors, lists, functions, and the ``NULL`` object. We also discuss the two common
compound objects in R, namely data frames and factors.

.. _sec-attributes:

Attributes
~~~~~~~~~~

.. index::
    pair: R function; attributes()
    pair: R function; attr()
    single: NULL

Before we discuss these objects, it is important to know that all objects
except the ``NULL`` object can have one or more attributes. Common attributes
are the ``names`` attribute which contains the element names, the ``class``
attribute which stores the name of the class of the object, and the ``dim``
attribute and (optionally) its ``dimnames`` companion to store the size of each
dimension (and the name of each dimension) of the object. For each object, the
``attributes()`` command will return a list with the attributes and their
values. The value of a specific attribute can be obtained using the ``attr()``
function. For example, ``attr(x, "class")`` will return the name of the class
of the object (or ``NULL`` if the attribute is not defined).

Vectors
~~~~~~~

.. index::
    single: atomic vectors

There are six basic vector types which are referred to as the *atomic vector
types*. These are:

logical:
    a boolean value (for example: ``TRUE``)

integer:
    an integer value (for example: ``1``)

double:
    a real number (for example: ``1.5``)

character:
    a character string (for example: ``"foobar"``)

complex:
    a complex number (for example: ``1+2i``)

raw:
    uninterpreted bytes (forget about this one)

These vectors have a length and can be indexed using ``[`` as the following sample
R session demonstrates:

.. code-block:: rout

    > x <- 2
    > length(x)
    [1] 1
    > y <- c(2, 3)
    > y[2]
    [1] 3
    > 
    
.. index::
    single: NA

As you can see, even single numbers are vectors with length equal to one.
Vectors in R can have missing values that are represented as ``NA``. Because all
elements in a vector must be of the same type (i.e. logical, double, int, etc.)
there are multiple types of ``NA``. However, the casual R user will generally
not be concerned with the different types for ``NA``.

.. code-block:: rout

    > x <- c(1, NA, 3)
    > x
    [1]  1 NA  3
    > y <- as.character(NA)
    > y
    [1] NA
    > typeof(NA) # default type of NA is logical
    [1] "logical"
    > typeof(y) # but we have coerced 'y' to a character vector
    [1] "character"
    > 

R's ``typeof()`` function returns the internal type of each object. In the
example above, ``y`` is a character vector.

Factors
~~~~~~~

.. index::
    single: factors
    pair: R function; as.factor()

Factors are one of R's compound data types. Internally, they are represented by
integer vectors with a ``levels`` attribute. The following sample R session
creates such a factor from a character vector:

.. code-block:: rout

    > x <- sample(c("A", "B", "C"), size = 10, replace = TRUE)
    > x
     [1] "C" "B" "B" "C" "A" "A" "B" "B" "C" "B"
    > as.factor(x)
     [1] C B B C A A B B C B
    Levels: A B C
    > 

Internally, the factor in this example is stored as an integer vector ``c(3, 2,
2, 3, 1, 1, 2, 2, 3, 2)`` which are the indices of the letters in the character
vector ``c(A, B, C)`` stored in the ``levels`` attribute. 

.. _sec-lists:

Lists
~~~~~

Lists are R's go-to structures for representing data structures. They can
contain multiple elements, each of which can be of a different type. Record-like
structures can be created by naming each element in the list. The ``lm()``
function, for example, returns a list that contains many details about the
fitted linear model. The following R session shows the difference between a list
and a list with named elements:

.. code-block:: rout

    > l <- list("Jane", 23, c(6, 7, 9, 8))
    > l
    [[1]]
    [1] "Jane"

    [[2]]
    [1] 23

    [[3]]
    [1] 6 7 9 8

    > l <- list(name = "Jane", age = 23, scores = c(6, 7, 9, 8))
    > l
    $name
    [1] "Jane"

    $age
    [1] 23

    $scores
    [1] 6 7 9 8

.. index::
    single: generic vectors

In R, lists are also known as *generic vectors*. They have a length that is
equal to the number of elements in the list.

Data frames
~~~~~~~~~~~

Data frames are one of R's compound data types. They are lists of vectors,
factors and/or matrices, all having the same length. It is one of the most
important concepts in statistics and has equivalent implementations in SAS_ and
SPSS_.

.. index::
    pair: R function; data.frame()
    pair: R function; is.list()

The following sample R session shows how a data frame is constructed, what its
attributes are and that it is indeed a list:

.. code-block:: rout

    > df <- data.frame(x = seq(5), y = runif(5))
    > df
      x         y
    1 1 0.8773874
    2 2 0.4977048
    3 3 0.6719721
    4 4 0.2135386
    5 5 0.3834681
    > class(df)
    [1] "data.frame"
    > attributes(df)
    $names
    [1] "x" "y"

    $row.names
    [1] 1 2 3 4 5

    $class
    [1] "data.frame"

    > is.list(df)
    [1] TRUE
    > 

.. _sec-matrices-and-arrays:

Matrices and arrays
~~~~~~~~~~~~~~~~~~~

.. index::
    pair: R function; dim()

Besides one-dimensional vectors, R also knows two other classes to represent
array-like data types: ``matrix`` and ``array``. A matrix is simply an atomic
vector with a ``dim`` attribute that contains a numeric vector of length two:

.. code-block:: rout

    > x <- seq(9)
    > class(x)
    [1] "integer"
    > dim(x) <- c(3, 3)
    > class(x)
    [1] "matrix"
    > x
         [,1] [,2] [,3]
    [1,]    1    4    7
    [2,]    2    5    8
    [3,]    3    6    9
    > 

Likewise, an array is also a vector with a ``dim`` attribute that contains a
numeric vector of length greater than two:

.. code-block:: rout

    > y <- seq(8)
    > dim(y) <- c(2,2,2)
    > class(y)
    [1] "array"
    > 

The example with the matrix shows that the elements in an array are stored in
`column-major order`_ which is important to know when we want to access R
arrays from a Java application.

.. note::

    In both examples for the ``matrix`` and ``array`` objects, the ``class()``
    function derives the class from the fact that the object is an atomic vector
    with the ``dim`` attribute set. Unlike data frames, these objects do not
    have a ``class`` attribute.

.. _column-major order: http://en.wikipedia.org/wiki/Row-major_order#Column-major_order

Overview of Renjin's type system
--------------------------------

.. index::
    pair: R function; typeof()

Renjin has corresponding classes for all of the R object types discussed in the
section :ref:`sec-java-developer-guide-to-r-objects`. Table
:ref:`tab-renjin-type-classes` summarizes these object types and their Java
classes. In R, the object type is returned by the ``typeof()`` function.


.. _tab-renjin-type-classes:

.. table:: Renjin's Java classes for common R object types

    =====================   =======================
    R object type           Renjin class
    =====================   =======================
    logical                 LogicalVector
    integer                 IntVector
    double                  DoubleVector
    character               StringVector
    complex                 ComplexVector
    raw                     RawVector
    list                    ListVector
    function                Function
    environment             Environment
    NULL                    Null
    =====================   =======================


There is a certain hierarchy in Renjin's Java classes for the different object
types in R. Figure :ref:`fig-renjin-type-system` gives a full picture of all
classes that make up Renjin's type system. These classes are contained in the
*org.renjin.sexp* Java package. The vector classes listed in table
:ref:`tab-renjin-type-classes` are in fact abstract classes that can have
different implementations. For example, the ``DoubleArrayVector`` (not shown in
the figure) is an implementation of the ``DoubleVector`` abstract class. The
:java:ref:`SEXP`, :java:ref:`Vector`, and ``AtomicVector`` classes are all Java
interfaces.

.. note::

    Renjin does not have classes for all classes of objects that are know to
    (base) R. This includes objects of class ``matrix`` and ``array`` which are
    represented by one of the ``AtomicVector`` classes and R's compound objects
    ``factor`` and ``data.frame`` which are represented by an ``IntVector`` and
    :java:ref:`ListVector` respectively.


.. _fig-renjin-type-system:

.. figure:: /images/renjin-class-hierarchy.png

    Hierarchy in Renjin's type system


.. _sec-pulling-data-from-r-into-java:

Pulling data from R into Java
-----------------------------

Now that you have a good understanding of both R's object types and how these
types are mapped to Renjin's Java classes, we can start by pulling data from R
code into our Java application. A typical scenario is one where an R script
performs a calculation and the result is pulled into the Java application for
further processing.

Using the Renjin Script Engine as introduced in the :doc:`introduction`, we can
store the result of a calculation from R into a Java object. By default, the
``eval()`` method of :java:ref:`javax.script.ScriptEngine` returns an
:java:ref:`Object <java.lang.Object>`, i.e. Java's object superclass. We can
always cast this result to a :java:ref:`SEXP` object. The following Java
snippet shows how this is done and how the :java:ref:`Object.getClass()
<java.lang.Object.getClass()>` and :java:ref:`Class.getName()
<java.lang.Class.getName()>` methods can be used to determine the actual class
of the R result:

.. code-block:: java

    // evaluate Renjin code from String:
    SEXP res = (SEXP)engine.eval("a <- 2; b <- 3; a*b");

    // print the result to stdout:
    System.out.println("The result of a*b is: " + res);      
    // determine the Java class of the result:
    Class objectType = res.getClass();
    System.out.println("Java class of 'res' is: " + objectType.getName());
    // use the getTypeName() method of the SEXP object to get R's type name:
    System.out.println("In R, typeof(res) would give '" + res.getTypeName() + "'");

This should write the following to the standard output::

    The result of a*b is: 6.0
    Java class of 'res' is: org.renjin.sexp.DoubleArrayVector
    In R, typeof(res) would give 'double'

As you can see the :java:ref:`getTypeName` method of the :java:ref:`SEXP` class
will return a String object with R's name for the object type.

.. note::

    Don't forget to import ``org.renjin.sexp.*`` to make Renjin's type classes
    available to your application.

In the example above we could have also cast R's result to a *DoubleVector*
object:

.. code-block:: java

    DoubleVector res = (DoubleVector)engine.eval("a <- 2; b <- 3; a*b");

or you could cast it to a *Vector*:

.. code-block:: java

    Vector res = (Vector)engine.eval("a <- 2; b <- 3; a*b");

You can't cast R integer results to a ``DoubleVector``: the following snippet
will throw a :java:ref:`ClassCastException <java.lang.ClassCastException>`:

.. code-block:: java

    // use R's 'L' suffix to define an integer:
    DoubleVector res = (DoubleVector)engine.eval("1L");
    
Accessing individual elements of vectors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now that we know how to pull R objects into our Java application we want to work
with these data types in Java. In this section we show how individual elements
of the Vector objects can be accessed in Java.

As you know, each vector type in R, and thus also in Renjin, has a length which
can be obtained with the ``length()`` method. Individual elements of a vector
can be obtained with the ``getElementAsXXX()`` methods where ``XXX`` is one of
``Double``, ``Int``, ``String``, ``Logical``, and ``Complex``. The following
snippet demonstrates this:

.. code-block:: java

    Vector x = (Vector)engine.eval("x <- c(6, 7, 8, 9)");
    System.out.println("The vector 'x' has length " + x.length());
    for (int i = 0; i < x.length(); i++) {
        System.out.println("Element x[" + (i + 1) + "] is " + x.getElementAsDouble(i));
    }

This will write the following to the standard output::

    The vector 'x' has length 4
    Element x[1] is 6.0
    Element x[2] is 7.0
    Element x[3] is 8.0
    Element x[4] is 9.0

As we have seen in the :ref:`sec-lists` section above, lists in R are also known
as *generic vectors*, but accessing the individual elements and their elements
requires a bit more care. If an element (i.e. a vector) of a list has length
equal to one, we can access this element directly using one of the
``getElementAsXXX()`` methods. For example:

.. code-block:: java

    ListVector x =
        (ListVector)engine.eval("x <- list(name = \"Jane\", age = 23, scores = c(6, 7, 8, 9))");
    System.out.println("List 'x' has length " + x.length());
    // directly access the first (and only) element of the vector 'x$name':
    System.out.println("x$name is '" + x.getElementAsString(0) + "'");
    
which will result in::

    List 'x' has length 3
    x$name is 'Jane'

being printed to standard output. However, this approach will not work for the
third element of the list as this is a vector with length greater than one.
The preferred approach for lists is to get each element as a :java:ref:`SEXP`
object first and then to handle each of these accordingly. For example:

.. code-block:: java

    DoubleVector scores = (DoubleVector)x.getElementAsSEXP(2);
    
.. _sec-dealing-with-matrices:

Dealing with matrices
~~~~~~~~~~~~~~~~~~~~~

As described in the section :ref:`sec-matrices-and-arrays` above, matrices are
simply vectors with the ``dim`` attribute set to an integer vector of length
two. In order to identify a matrix in Renjin, we need to therefore check for
the presence of this attribute and its value. Since any object in R can have
one or more attributes, the :java:ref:`SEXP` interface defines a number of
methods for dealing with attributes. In particular, :java:ref:`hasAttributes`
will return ``true`` if there are any attributes defined in an object and
:java:ref:`getAttributes` will return these attributes as a
:java:ref:`AttributeMap`.

.. code-block:: java

    Vector res = (Vector)engine.eval("matrix(seq(9), nrow = 3)");
    if (res.hasAttributes()) {
        AttributeMap attributes = res.getAttributes();
        Vector dim = attributes.getDim();
        if (dim == null) {
            System.out.println("Result is a vector of length " +
                res.length());
    
        } else {
            if (dim.length() == 2) {
                System.out.println("Result is a " +
                    dim.getElementAsInt(0) + "x" +
                    dim.getElementAsInt(1) + " matrix.");
            } else {
                System.out.println("Result is an array with " +
                    dim.length() + " dimensions.");
            }
        }
    }
    
Output:

.. code-block:: none

    Result is a 3x3 matrix.
    
For convenience, Renjin includes a wrapper class ``Matrix`` that provides
easier access to the number of rows and columns.

.. index::
    pair: R function; matrix()

Example:

.. code-block:: java

    // required import(s):
    import org.renjin.primitives.matrix.*;

    Vector res = (Vector)engine.eval("matrix(seq(9), nrow = 3)");
    try {
        Matrix m = new Matrix(res);
        System.out.println("Result is a " + m.getNumRows() + "x"
            + m.getNumCols() + " matrix.");
    } catch(IllegalArgumentException e) {
        System.out.println("Result is not a matrix: " + e);
    }
    
Output:

.. code-block:: none

    Result is a 3x3 matrix.

.. _sec-dealing-with-lists-and-data-frames:

Dealing with lists and data frames
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The :java:ref:`ListVector` class contains several convenience methods to access
a list's components from Java. For example, we can the extract the components
from a fitted linear model using the name of the element that contains those
components. For example:

.. code-block:: java

    ListVector model = (ListVector)engine.eval("x <- 1:10; y <- x*3; lm(y ~ x)");
    Vector coefficients = model.getElementAsVector("coefficients");
    // same result, but less convenient:
    // int i = model.indexOfName("coefficients");
    // Vector coefficients = (Vector)model.getElementAsSEXP(i);
    
    System.out.println("intercept = " + coefficients.getElementAsDouble(0)); 
    System.out.println("slope = " + coefficients.getElementAsDouble(1)); 
    
Output:

.. code-block:: none

    intercept = -4.4938668397781774E-15
    slope = 3.0
    

        
.. _sec-dealing-with-errors-in-the-R-code:

Handling errors generated by the R code
---------------------------------------

Up to now we have been able to execute R code without any concern for possible
errors that may occur when the R code is evaluated. There are two common
exceptions that may be thrown by the R code: 

.. index::
    pair: R function; stop()
    single: exceptions

1. :java:ref:`ParseException`: an exception thrown by Renjin's R parser due to a syntax error and 
2. :java:ref:`EvalException`: an exception thrown by Renjin when the R code generates an error condition, for example by the ``stop()`` function.

Here is an example which catches an exception from Renjin's parser:

.. code-block:: java

    // required import(s):
    import org.renjin.parser.ParseException;

    try {
        engine.eval("x <- 1 +/ 1");
    } catch (ParseException e) {
        System.out.println("R script parse error: " + e.getMessage());
    }
    
Output:

.. code-block:: none

    R script parse error: Syntax error at line 1 char 0: syntax error, unexpected '/'

And here's an example which catches an error condition thrown by the R interpreter:

.. code-block:: java

    // required import(s):
    import org.renjin.eval.EvalException;

    try {
        engine.eval("stop(\"Hello world!\")");
    } catch (EvalException e) {
        // getCondition() returns the condition as an R list:
        Vector condition = (Vector)e.getCondition();
        // the first element of the string contains the actual error message:
        String msg = condition.getElementAsString(0);
        System.out.println("The R script threw an error: " + msg);
    }
    
Output:

.. code-block:: none

    The R script threw an error: Hello world!

:java:ref:`EvalException.getCondition()` is required to pull the condition
message from the R interpreter into Java.
    
    
.. _sec-pushing-data-from-java-to-r:

Pushing data from Java to R
---------------------------

Like many dynamic languages, R scripts are evaluated in the context of an
environment that looks a lot like a dictionary. You can define new variables in
this environment using the :java:ref:`javax.script` API. This is achieved using
the :java:ref:`ScriptEngine.put()
<javax.script.ScriptEngine.put(java.lang.String, java.lang.Object)>` method.

Example:

.. code-block:: java

    engine.put("x", 4);
    engine.put("y", new double[] { 1d, 2d, 3d, 4d });
    engine.put("z", new DoubleArrayVector(1,2,3,4,5));
    engine.put("hashMap", new java.util.HashMap());
    // some R magic to print all objects and their class with a for-loop:
    engine.eval("for (obj in ls()) { " +
        "cmd <- parse(text = paste('typeof(', obj, ')', sep = ''));" +
        "cat('type of ', obj, ' is ', eval(cmd), '\\n', sep = '') }");
    
Output:

.. code-block:: none

    type of hashMap is externalptr
    type of x is integer
    type of y is double
    type of z is double
    
Renjin will implicitly convert primitives, arrays of primitives and
:java:ref:`String` instances to R objects. Java objects will be wrapped as R
``externalptr`` objects. The example also shows the use of the
``DoubleArrayVector`` constructor to create a double vector in R. You see
that we managed to put a Java :java:ref:`java.util.HashMap` object into the
global environment of the R session: this is the topic of the chapter
:ref:`chap-importing-java-classes-into-R-code`.

.. _SAS: http://www.sas.com
.. _SPSS: http://www.ibm.com/software/analytics/spss/

