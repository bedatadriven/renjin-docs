.. _sec-thread-safety:

Thread-Safety
==========================

R code must always be evaluated in the context of a Session, which carries certain
state, such as the Global Environment the list of loaded packages, global options, and the
state of the random number generator.

Sessions are not thread-safe in the sense that two different R expressions cannot be
evaluated concurrently within the same Session.

When using GNU R, a new R Session begins when the interpreter is started in a process,
either from the command line, or via REngine. Because of the way that GNU R is implemented,
every R Session must have its own process, and so you can not evaluate two R expressions
concurrently in the same process.

Renjin is implemented differently, with the goal of being able to run multiple Sessions 
within the same process or the same JVM. 

Every new instance of RenjinScriptEngine has its own independent Session. A single Session
cannot execute multiple R scripts concurrently, but you can execute multiple R scripts 
concurrently within the same JVM, as long as each concurrent script has its own ScriptEngine.

If you want to execute several R scripts in parallel, you have a few options. 

Thread-Local ScriptEngine
-------------------------

You can use the standard :java:ref:`java.lang.ThreadLocal` class to maintain exactly
one ScriptEngine per thread. This approach is useful when a small number of long-running worker threads
each need to execute independent R scripts. This is the case for most Java Servlet containers, for example.

The following is a simple example based on the appengine-servlet_ example:

.. code-block:: java

    public class MyServlet extends HttpServlet {

        /**
         * Maintain one instance of Renjin for each request thread.
         */
        private static final ThreadLocal<ScriptEngine> ENGINE = new ThreadLocal<ScriptEngine>();

   
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

            ScriptEngine engine = ENGINE.get();
            if(engine == null) {
                // create a new engine for this thread
                RenjinScriptEngineFactory factory = new RenjinScriptEngineFactory();
                engine = factory.getScriptEngine();
                ENGINE.set(engine);
            }
            // evaluate a script using the engine reference
        } 

.. _appengine-servlet: https://github.com/bedatadriven/renjin-examples/blob/master/appengine-servlet/src/main/java/org/renjin/example/appengine/RenjinServlet.java#L44

ScriptEngine Pooling
--------------------

If your application, in constrast, uses a larger number of threads, or short-lived threads, it may
make more sense to use a pool of ScriptEngines that can be leased to worker threads as needed.

The `Apache Commons Pool`_ project provides a solid implementation that can be easily used to 
pool Renjin ScriptEngine sessions.

.. _`Apache Commons Pool`: https://commons.apache.org/proper/commons-pool/


