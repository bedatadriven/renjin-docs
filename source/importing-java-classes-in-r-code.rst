.. _chap-importing-java-classes-into-R-code:

Importing Java classes into R code
==================================

.. index::
    pair: Renjin function; import()

A true testament of the level of integration of Java and R in Renjin is the
ability to directly access (public!) Java classes and methods from R code.
Renjin introduces the ``import()`` function which adds a Java class to the
environment from which it is called. In the section
:ref:`sec-pushing-data-from-java-to-r` in the previous chapter we had already
seen how a Java class could be put into the global environment of the R
session.

Consider the following sample R script:

.. code-block:: r

    import(java.util.HashMap)

    # create a new instance of the HashMap class:
    ageMap <- HashMap$new()

    # call methods on the new instance:
    ageMap$put("Bob", 33)
    ageMap$put("Carol", 41)

    print(ageMap$size()) 

    age <- ageMap$get("Carol")
    cat("Carol is ", age, " years old.\n", sep = "")

    # Java primitives and their boxed types
    # are automatically converted to R vectors:
    typeof(age)  


As we showed in the :doc:`introduction`, we can execute this script using the
:java:ref:`java.io.FileReader` interface:

.. code-block:: java

    engine.eval(new java.io.FileReader("import_example.R"));
    
Output:

.. code-block:: none

    [1] 2
    Carol is 41 years old.
    
The first line in the output is the output from the ``print(ageMap$size())``
statement.

Bean classes
------------

For Java classes with accessor methods that conform to the getXX() and setXX()
Java bean convention, Renjin provides some special sauce to make access from R
more natural.

Take the following Java bean as an example:

.. code-block:: java

    package beans;

    public class Customer {
        private String name;
        private int age;

        public String getName() { return name; }
        public void setName(String name) { this.name = name; }

        public int getAge() { return age; }
        public void setAge(int age) { this.age = age; }
    }

You can construct a new instance of the ``Customer`` class and provide initial
values with named arguments to the constructor. For example:

.. code-block:: r

    import(beans.Customer)

    bob <- Customer$new(name = "Bob", age = 36)
    carol <- Customer$new(name = "Carol", age = 41)
    cat("'bob' is an ", typeof(bob), ", bob$name = ", bob$name, "\n", sep = "")
    # the original java methods are also available i.e. the following is equivalent
    cat("'bob' is an ", typeof(bob), ", bob$getName() = ", bob$getName(), "\n", sep = "")

If the previous R code is stored in a file ``bean_example.R`` then the
following Java code snippet runs this example:

.. code-block:: java

    // required import(s):
    import beans.Customer;

    engine.eval(new java.io.FileReader("bean_example.R"));
    
Output:

.. code-block:: none

    'bob' is an externalptr, bob$name = Bob
    'bob' is an externalptr, bob$getName() = Bob
