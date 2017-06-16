
Using Renjin as an R Package
==========================

.. index::
   single: REPL

Though Renjin's ultimate goal is to be a complete, drop-in replacement
for GNU R, in some cases you may want to run only part of your existing
R code with Renjin, from within GNU R.

For this use case, you can load Renjin as a package and evaluate
performance-sensitive sections of your code, without having to rewrite
the R code.

Prerequisites
-------------

You must have Java 7 installed or higher. For best performance, we recommend
using the latest version of Oracle or OpenJDK 8. The rJava package is also
required, but should be installed automatically.

   
Installation (R >= 3.4)
-----------------------

.. code-block:: R

   install.packages("https://nexus.bedatadriven.com/content/groups/public/org/renjin/renjin-gnur-package/0.8.2353/renjin-gnur-package-0.8.2353.tar.gz")


Installation (R < 3.3)
-----------------------

.. code-block:: R

   download.file("https://nexus.bedatadriven.com/content/groups/public/org/renjin/renjin-gnur-package/0.8.2353/renjin-gnur-package-0.8.2353.tar.gz", "renjin.tgz")
   install.packages("renjin.tgz", repos = NULL, type = "source")


Usage
-----

.. code-block:: R

   library(renjin)

   bigsum <- function(n) {
     sum <- 0
     for(i in seq(from = 1, to = n)) {
       sum <- sum + i
     }
     sum
   }
   bigsumc <- compiler::cmpfun(bigsum) # GNU R's byte code compiler

   system.time(bigsum(1e8)) 
   system.time(bigsumc(1e8))
   system.time(renjin(bigsum(1e8)))









