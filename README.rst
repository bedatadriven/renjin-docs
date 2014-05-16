Renjin developer documentation
==============================

This repository contains the sources for Renjin's developer documentation. The
documentation is created using `Sphinx`.

Requirements to build the documentation are:

* `Sphinx`, obviously.
* The `javasphinx` extension for Sphinx.
* LaTeX, if you want to create a PDF version of the documentation.

If you want to creat the HTML documentation using the lovely `RTD theme` you
will need to install this as well:

    pip install sphinx_rtd_theme

To start
    
    git clone https://github.com/bedatadriven/renjin-docs.git

To create the HTML version:

    make html

and to make the PDF version:

    make latexpdf

Copyright
---------

Copyright 2014, `BeDataDriven`.

.. _Sphinx: http://sphinx-doc.org/
.. _Read The Docs: http://www.readthedocs.org
.. _javasphinx: http://bronto.github.io/javasphinx/
.. _RTD theme: https://read-the-docs.readthedocs.org/en/latest/theme.html
.. _BeDataDriven: http://www.bedatadriven.com
