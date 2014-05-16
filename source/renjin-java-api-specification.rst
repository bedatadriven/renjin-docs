Renjin Java API specification
=============================

This chapter includes a selection of public classes and methods in some of
Renjin's Java packages which are most useful to users of Renjin looking to
exchange data between Java and R code.

org.renjin.sexp
---------------

The ``org.renjin.sexp`` package contains Renjin's classes that represent the
different object types in R.

.. java:package:: org.renjin.sexp

AttributeMap
~~~~~~~~~~~~

.. java:type:: class AttributeMap

    Renjin's implementation to store object attributes. See the
    :ref:`sec-attributes` section for a short introduction to attributes on
    objects in R.

    .. java:method:: SEXP getClassVector()

        See :java:ref:`hasClass` for an example.

        :return: a character vector of classes or ``null`` if no ``class`` attribute is defined

    .. java:method:: Vector getDim()

        :return: the ``dim`` attribute as a :java:ref:`Vector` or ``null`` if no dimension is defined

    .. java:method:: AtomicVector getNamesOrNull()

        :return: the ``names`` attribute as a :java:ref:`AtomicVector` or ``null`` if no names are defined

    .. java:method:: boolean hasClass()

        :return: ``true`` if the ``class`` attribute exists, ``false`` otherwise

        Example:

        .. code-block:: java

            Vector df = (Vector)engine.eval("df <- data.frame(x = seq(5), y = runif(5))");
            AttributeMap attributes = df.getAttributes();
            if (attributes.hasClass()) {
                System.out.println("Classes defined on 'df': " +
                    (Vector)attributes.getClassVector());
            }
            
        Output:

        .. code-block:: none

            Classes defined on 'df': "data.frame"

    .. java:method:: ListVector toVector()

        Convert the object's attributes to a list.

        :return: attributes as a :java:ref:`ListVector`
    
ListVector
~~~~~~~~~~

.. java:type:: class ListVector extends AbstractVector

    Renjin's class that represent R lists and data frames. Data frames are
    lists with the restriction that all elements, which are atomic vectors,
    have the same length.

    .. java:method:: SEXP getElementAsSEXP(int index)

        :param index: zero-based index

        :return: a ``SEXP`` that can be cast to a vector type

    .. java:method:: Vector getElementAsVector(String name)

        Convenience method to get the named element of a list. See the section
        :ref:`sec-dealing-with-lists-and-data-frames` for an example.

        :param name: element name as string

        :return: a :java:ref:`Vector`

    .. java:method:: int indexOfName(String name)

        :param name: element name as string

        :return: zero-based index of ``name`` in the names attribute, -1 if ``name`` is not present in the names attribute or if the names attribute is not set

    .. java:method:: boolean isElementNA(int index)

        Check if an element of a list is ``NA``.

        :param index: zero-based index

        :return: ``true`` if the element at position ``index`` is an ``NA``, false otherwise

Logical
~~~~~~~

.. java:type:: enum Logical

    A logical value in R can be ``TRUE``, ``FALSE``, or the logical ``NA``.

    .. java:method:: static Logical valueOf(boolean b)

        Turn a Java boolean into an R logical value.

        :param b: ``true`` or ``false``

        :return: R's ``TRUE`` or ``FALSE`` as Renjin's representation of a logical value

    .. java:method:: static Logical valueOf(int i)

        Turn an integer into an R logical value.

        :param i: an integer value

        :return: ``TRUE`` if ``i`` is 1, ``FALSE`` if ``i`` is 0, or (logical) ``NA`` otherwise

SEXP
~~~~

.. java:type:: interface SEXP

    Renjin's superclass for all objects that are mapped from R's object types.

    .. java:method:: AttributeMap getAttributes()

        Get the attributes of an object as a ``AttributeMap`` which is
        Renjin's way of working with object attributes. R stores attributes as
        pairlists which are essentially the same as *generic lists*, therefore
        these attributes can safely be stored in a list. Renjin provides a
        :java:ref:`toVector()` method to do just that.

        :return: the attributes for the object as a, possibly empty, :java:ref:`AttributeMap`

        Example:

        .. code-block:: java

            ListVector res = (ListVector)engine.eval(
                    "list(name = \"Jane\", age = 23, scores = c(6, 7, 8, 9))");
                // use ListVector.toString() method to display the list:
                System.out.println(res);
                if (res.hasAttributes()) {
                    AttributeMap attributes = res.getAttributes();
                    // convert the attribute map to something more convenient:
                    ListVector attributeList = attributes.toVector();
                    System.out.println("The list has "
                        + attributeList.length() + " attribute(s)");
                }
            
        Output:

        .. code-block:: none

            list(name = "Jane", age = 23.0, scores = c(6, 7, 8, 9))
            The list has 1 attribute(s)

    .. java:method:: String getTypeName()

        Get the type of the object as it is known in R, i.e. the result of R's
        ``typeof()`` function.

        :return: the object type as a string

        Example:

        .. code-block:: java

            Vector x = (Vector)engine.eval("NA");
            System.out.println("typeof(NA) = " + x.getTypeName());
            
        Output:

        .. code-block:: none

            typeof(NA) = logical
    
    .. java:method:: boolean hasAttributes()

        Check for the presence of attributes. See :java:ref:`getAttributes` for
        an example.

        :return: ``true`` if the object has at least one attribute, ``false`` otherwise

    .. index::
        pair: R function; length()
        pair: Java; length()

    .. java:method:: int length()

        Get the length of the object. All objects in R have a length and this
        method gives the same result as R's ``length()`` function. Functions
        always have length 1 and the ``NULL`` object always has length 0. The
        length of an environment is equal to the number of objects inside the
        environment.

        :return: length of the vector as an integer

Vector
~~~~~~

.. java:type:: interface Vector extends SEXP

        An interface which represents all vector object types in R: atomic
        vectors and *generic vectors* (i.e. :ref:`sec-lists`).

    .. java:method:: double getElementAsDouble(int index)

        :param index: zero-based index

        :return: the element at ``index`` as a double, converting if necessary; ``NaN`` if no conversion is possible

        Example:

        .. code-block:: java

            // create a string vector in R:
            Vector x = (Vector)engine.eval("c(\"foo\", \"bar\")");
            double x1 = x.getElementAsDouble(0);
            if (Double.isNaN(x1)) {
                System.out.println("Result is NaN");
            }
            String s = x.getElementAsString(0);
            System.out.println("First element of result is " + s);
            // call the toString() method of the underlying StringArrayVector:
            System.out.println("Vector as defined in R: " + x);
            
        Output:

        .. code-block:: none

            Result is NaN
            First element of result is foo
            Vector as defined in R: c(foo, bar)

        .. note::

            All of the classes that implement the :java:ref:`Vector` interface
            have a ``toString()`` method that will display (a short form of)
            the content of the vector. This method is provided for debugging
            purposes only.

    .. java:method::  int getElementAsInt(int index)

        :param index: zero-based index

        :return: the element at ``index`` as an integer, converting if necessary; ``NaN`` if no conversion is possible
        
    .. java:method:: String getElementAsString(int index)

        :param index: zero-based index

        :return: the element at ``index`` as a string
   
    .. java:method:: Logical getElementAsLogical(int index)

        :param index: zero-based index

        :return: the element at ``index`` as Renjin's representation of a boolean value


org.renjin.primitives.matrix
----------------------------

.. java:package:: org.renjin.primitives.matrix

Matrix
~~~~~~

.. java:type:: class Matrix

    Wrapper class for a :java:ref:`Vector` with two dimensions. Simplifies
    interaction with R matrices from Java code.

    .. java:constructor:: Matrix(Vector vector)

        Constructor for creating a matrix from a :java:ref:`Vector`. Checks if
        the dimension attribute is present and has length 2, throws an
        :java:ref:`IllegalArgumentException` if not. See the section
        :ref:`sec-dealing-with-matrices` for an example.

        :param vector: a vector with two dimensions

        :throws IllegalArgumentException: if the ``dim`` attribute of ``vector`` does not have length 2

    .. java:method:: int getNumRows()

        :return: number of rows in the matrix

    .. java:method:: int getNumCols()

        :return: number of columns in the matrix

Exceptions
----------

.. index::
    single: exceptions
    
.. java:package:: org.renjin.parser

.. java:type:: class ParseException extends RuntimeException

    An exception thrown by Renjin's parser when there is an error in parsing R
    code, usually due to a syntax error. See
    :ref:`sec-dealing-with-errors-in-the-R-code` for an example that catches
    this exception.

.. java:package:: org.renjin.eval

.. java:type:: class EvalException extends RuntimeException

    An exception thrown by Renjin's interpreter when the R code generates an
    error condition, e.g. by the ``stop()`` function. See
    :ref:`sec-dealing-with-errors-in-the-R-code` for an example that catches
    this exception.

    .. java:method:: SEXP getCondition()

        :return: a :java:ref:`SEXP` that is a list with a single named element ``message``. Use :java:ref:`getElementAsString()` to obtain the actual error message.

