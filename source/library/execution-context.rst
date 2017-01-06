.. _sec-execution-context:

Customizing the Execution Context
=================================

R code must always be evaluated in the context of a Session, with tracks the global environment,
which packages have been loaded, etc.

Each new ``ScriptEngine`` instance has it's own independent Session, and for each Session,
Renjin allows you to customize the environment in which the R scripts are evaluated with regard to:

.. contents::  :local:


The ``SessionBuilder`` object provides an API for creating a customized Renjin
ScriptEngine instance:

.. code-block:: java

    import javax.script.*;
    import org.renjin.eval.*;
    import org.renjin.script.*;

    Session session = new SessionBuilder()
      .withDefaultPackages()
      .build();

    RenjinScriptEngineFactory factory = new RenjinScriptEngineFactory();
    RenjinScriptEngine engine = factory.getScriptEngine(session);

The sections below outline how the methods of the SessionBuilder object 
can be used to customize the execution context.



File System
-----------

The R language provides many builtin functions that allow scripts to interact with 
the file system, such as ``getwd()``, ``setwd()``, ``file()``, ``list.files()`` etc. 

In some contexts, however, direct access to the file system may not be appropriate. For example:

* You may want to limit the ability of a script to write to the file system.
* You may want to run an R script that expects data on the local file system, but
  redirect calls to ``file()`` to an alternate data source, such as a network resource or
  a database.

For this reason, Renjin mediates all calls to R file system functions through the
`Apache Commons Virtual File System (VFS) <https://commons.apache.org/proper/commons-vfs/>`_ 
library. 

You can provide your own ``FileSystemManager`` instance to ``SessionBuilder`` configured for 
your particular use case.

The following example demonstrates how a ScriptEngine instance is configured by default:

.. code-block:: java

    DefaultFileSystemManager fsm = new DefaultFileSystemManager();
    fsm.addProvider("jar", new JarFileProvider());
    fsm.addProvider("file", new LocalFileProvider());
    fsm.addProvider("res", new ResourceFileProvider());
    fsm.addExtensionMap("jar", "jar");
    fsm.setDefaultProvider(new UrlFileProvider());
    fsm.setBaseFile(new File("/"));
    fsm.init();

    Session session = new SessionBuilder()
      .withDefaultPackages()
      .setFileSystemManager(fsm)
      .build();

    RenjinScriptEngineFactory factory = new RenjinScriptEngineFactory();
    RenjinScriptEngine engine = factory.getScriptEngine(session);



The ``renjin-appengine`` module provides a more complex example. 
There, the `AppEngineContextFactory`_ class prepares a FileSystemManager that is configured
with a LocalFileProvider subclass that provides read-only acesss to the servlet's directory.
This allows R scripts access to "/WEB-INF/data/model.R", which is translated into the absolute
path at runtime.

.. _AppEngineContextFactory: https://github.com/bedatadriven/renjin/blob/87518a254c0d67788aa36e0ecb4038d25a6d9384/appengine/src/main/java/org/renjin/appengine/AppEngineContextFactory.java#L88-L88

Package Loading
---------------

In contrast to GNU R, which always loads packages from the local file system, Renjin's
package loading mechanism is customizable in order to support different use cases:

* When used interactively, analysts expect to be able download and run packages interactively
  from CRAN or BioConductor.
* When embedding specific R code in a web application, R packages can be declared as dependencies 
  in Maven, Gradle, or SBT, and shipped along with the application just as any other JVM dependency.
* When allowing users to execute arbitrary R code in your application, you may want to limit 
  R packages to some approved subset and load from an internal repository.

For this reason, Renjin mediates all package loading, such as calls to ``library()`` or to ``require()``
through a ``PackageLoader`` interface. This allows the application executing R code to choose
an appropriate implementation.

Renjin itself provides two PackageLoader implementations:

* The ``ClasspathPackageLoader``, which is the default for ScriptEngines and only loads packages that
  are already on the classpath.
* The ``AetherPackageLoader``, which will download packages on demand from a remote Maven repository. 
  This is used by the Renjin interactive REPL.

If you are embedding Renjin in your application and want packages to be loaded on demand, then you can 
configure SessionBuilder with an instance of an ``AetherPackageLoader``.

The following example shows to add this dynamic behavior to a Renjin ScriptEngine, and adds an 
additional, internal Maven repository that is used to resolve packages:

.. code-block:: java

    RemoteRepository internalRepo = new RemoteReocs.pository.Builder(
        "internal", /* id */ 
        "default",  /* type */
        "https://repo.acme.com/content/groups/public/").build();
    
    List<RemoteRepository> repositories = new ArrayList<>();
    repositories.add(internalRepo);
    repositories.add(AetherFactory.renjinRepo());
    repositories.add(AetherFactory.mavenCentral());
    
    ClassLoader parentClassLoader = getClass().getClassLoader();
    
    AetherPackageLoader loader = new AetherPackageLoader(parentClassLoader, repositories);

    Session session = new SessionBuilder()
        .withDefaultPackages()
        .setPackageLoader(loader)
        .build();


You can also provide your own implementation of ``PackageLoader`` which resolves calls to 
``import()`` and ``require()`` in any other way that meets your needs.

Class Loading
-------------

When R packages depend on JVM classes by using Renjin's ``importClass()`` directive, the JVM class
is loaded indirectly via the Session's ``PackageLoader`` interface.

However, R scripts can also load JVM classes on an ad-hoc basis using the ``import(com.acme.MyClass)`` function.

Such classes are loaded not through the ``PackageLoader`` mechanism but through the Session object's own
``ClassLoader`` instance. This can also be set through the SessionBuilder object:

.. code-block:: java


    URLClassLoader classLoader = new URLClassLoader(
        new URL[] {
            new File("/home/alex/my_dir_with_jars").toURI().toURL(),
            new File("/home/alex/my_other_dir_with_jars").toURI().toURL()
        });
    
    Session session = new SessionBuilder()
        .setClassLoader(classLoader)
        .build();










