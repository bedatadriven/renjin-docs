# javasphinx (javaext)
The [javasphinx](https://github.com/bronto/javasphinx) Sphinx extension has been discontinued and 
is now archived and read-only. Sphinx version 1.8 and later introduced breaking changes to the extension api
(notably the sphinx.locale which used to have an l_() translation method which is now just _()).

For this reason there is nowhere to submit a patch for javasphinx but our docs depend on it to build.
Hence, a modified, patched version of javasphinx is included here as javaext to not conflict with 
an existing installation of javasphinx (which would conflict with our local one).