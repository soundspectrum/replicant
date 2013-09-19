Replicant
=========

![Rachael Replicant](https://raw.github.com/soundspectrum/replicant/master/replicant.jpg)
*Do you like our owl?*
~ Rachael, *Blade Runner* (1982)

What is Replicant?
------------------

Replicant is a commercial-grade, LLVM-based Python compiler and execution environment with a focus on performance, no-GIL threading, embeddability, and extensibility. Replicant originates from the belief that software designers should be free to choose the concurrency model most appropriate for their needs without costly performance trade-offs. Replicant provides:

* Restriction-free multi-threading ("free threading").

* A versatile, portable, and high-performance multi-threaded garbage collector.

* An implementation built atop LLVM, an industry standard next-generation compiler infrastructure.

* Key features for developers who want to embed/extend Python in their commercial applications.

* Commercial-grade implementation standards of compartmentalization, encapsulation, and reliability.



Introduction
------------

In an era where high-powered, multi-core systems are available on every desktop, laptop, and now even on smartphones, it's increasingly clear that language implementations should support real multi-threading capabilities. Multi-threading is a viable and popular approach in many other languages, and it's an area where Python has typically had to apologize.   Historically, Python code which requires concurrent execution uses fork() and interprocess communication (IPC), but this approach does not always yield the best performance for all workloads.  For example, IPC requires serialization of data structures, which can be cost prohibitive when data structures are intricate or unwieldy. Shared memory between forked processes may be an alternative to serialization in some cases, but this only helpful for large flat data structures, and isn't possible for particular classes of algorithms. Interprocess shared memory solutions also tend to complicate implementation since message passing is required to setup, communicate, and synchronize access to a shared memory region.

The standard CPython threading module offers support for threads, but there are significant caveats when performance is paramount. Specifically, since a global interpreter lock (GIL) is required to execute more than one thread safely in CPython, it's well understood that Python threads cannot execute even perfectly independent code concurrently (such as two threads each incrementing a non-shared integer value). Although C extension modules can release the GIL when the remaining work can be performed independently, in practice this requires careful extension module design and doesn't solve the GIL performance drawbacks when executing pure Python code.

In addition to the drawbacks associated with the presence of the GIL, CPython's use of reference counting has unfortunate side effects.  For example, when Python code uses multiple processes for concurrent execution, reference count updates incur copy-on-write costs on shared memory pages.  Additionally, reference count loops/cycles are possible, forcing the runtime to incur additional expenses associated with reference cycle analysis.  Finally, the code mechanics of any reference counting API are less attractive than a garbage collected environment. 

Replicant's primary objectives are to provide a Python front-end and runtime environment that:

 * Offers fully independent interpreter instances to run in a "free threaded" fashion.

 * Provides a modern, high performance concurrent garbage collector.

 * Leverages LLVM, a modern compiler infrastructure with enterprise-class industry support. LLVM offers desirable performance characteristics "out of the box" and provides an ideal framework for novel and noteworthy performance-related future work (e.g., type markup and inference subsystems, function specializers, JIT-able Python, etc.).

Project Codebase
----------------

Replicant is being developed with the resources of SoundSpectrum, a small music visualization software company with just enough resources to devote to a project originating from a passion and vision to see the Python implementation taken to a higher level (see the 'Project Background' section). While SoundSpectrum is interested in sharing its work on Replicant in ways consistent with open source ideals, SoundSpectrum is also interested receiving a fair share of the fruits of the resources spent and the risks taken while designing and developing Replicant. So until Replicant's future and performance becomes more clear, SoundSpectrum is deferring public release of the Replicant codebase until at least Spring 2014, or until SoundSpectrum has otherwise finished exploring its licensing options.

Project Status
--------------

As of September 2013, we have implemented research prototypes of the following subsystems:

 * A compiler front-end which reads Python input source and emits LLVM assembly
   text. The input AST is subjected to a series of transformations which lower
   the input code to a flattened IR which is more suitable for code
   generation. The resulting LLVM code relies heavily on the C runtime library
   and can be thought of as essentially a low-level C program. The
   implementation is far from feature-complete with respect to language support,
   but there is support for most forms of module, class, and function
   definitions, as well as core control flow constructs.

   The current compiler implementation is in Haskell, but a Python rewrite is
   planned as soon as the proof of concept benchmarks have met their
   objectives. We fully intend for Replicant to be able to bootsrap itself.

 * The low-level RObject runtime layer (written in C with some C++) which provides:

   * Data structure definitions for boxed Python values (RObjects).

   * A low-level API for performing operations on RObjects (binary operations, invocation of callables, etc.). Accesses to RObjects are currently synchronized via a straightforward multiple-reader-single-writer concurrency scheme.

   * An preliminary implementation of a concurrent mark-and-sweep garbage collector.

   * Execution context and thread management APIs.

 * An implementation sketch of the Replicant API and C++ support layer, which offers high-level support for: 

   * Loading, compiling, and executing Python modules and functions.

   * Various wrappers around RObjects for ease, safety, and idiomatic use. See the roadmap and future work sections below for upcoming activities and goals.


Roadmap
-------

Replicant has undergone several major design iterations since its inception in
March 2012.  Nearly two years later, with the multi-threaded GC and lowest level
C layers taking shape, early benchmarks demonstrate Replicant successfully
executing certain categories of vanilla Python code, such as iterative algebraic
computation. Since showing does more than telling, we will demonstrate
Replicant's performance advantages in as many exciting and convincing ways as
possible at PyCon 2014.  Our PyCon 2014 live demos will consist of:

* A realtime fractal animation that runs in single thread mode (benchmarked
  against CPython), and the same animation in multi-threaded mode on a 4 or 8
  core machine.

* A multi-threaded web server (designed for comparison against a forking web server).

* A realtime demo that shows Replicant offering itself as a BSP (Bulk
  Synchronous Parallel) solution, a class of "big data" computation which causes
  IPC/forking models to struggle when the relative cost of data serialization is
  high.

As intuition might suggest, the amount of Python code for the above demos above
are on the order of a page since their implementation relies on conventional
multi-threading idioms.

As of September 2013, work is focused on improving the performance of the GC
implementation, which borrows some notions from read-copy-update (RCU)
approaches (as opposed to relying on heavyweight OS synchronization).  For
example, Replicant executes the "straightline" Mandelbrot fractal code faster
than CPython, but a version of the benchmark which uses functions to encapsulate
code currently decreases execution performance significantly because of some
deficiencies in the GC implementation.  We are excited about the performance
gains we hope to obtain once we address the parts of GC which, until now, have
been provisional in getting it up and running.

With Replicant's design principles firmly established and presented in Spring
2014, the compiler front-end will be completely rewritten in Python (the
research prototype is currently implemented in Haskell), with the expectation
that it will eventually be able to bootstrap itself.  As we transition the
compiler from the Haskell prototype to a more Python-friendly implementation,
the C/C++ runtime and support layers will continue to evolve, adding support for
more language features and standard library support. By PyCon 2015, we
anticipate the Replicant roadmap to be fully clear, having completed the lion's
share of language support activities and having performed a bulk of the standard
library implementation/porting activities.

As the Replicant roadmap solidifies and the implementation matures, we will be
able to explore additional features and capabilities which may enable novel and
interesting uses of Replicant.

For example, we plan to experiment with "sandboxing" of OS calls into an
abstract set of "OS interfaces" that provide any and all calls into the OS (file
system, synchronization, network, threads, time, processes, etc).  The set of OS
interfaces applied to a Replicant execution context will naturally default to
the set of C++ classes shipped with Replicant for the host platform (e.g. POSIX,
Win32, Linux), but offer an important level of abstraction for developers with
sensitive security/separation needs.  For example, suppose that a security
evaluation of a piece of classified software requires that it contains no calls
into OS network APIs.  Replicant can simply be set to use the stub OS network
interface in lieu of the standard OS network interface for the host platform,
trapping certain calls.  Or suppose a financial institution requires that a
particular utility records all file activity into an metalog.  In Replicant,
this could be done at a low-level by overriding OS interfaces to additionally
write out the appropriate metadata.  In summary, Replicant sandboxing will offer
powerful and airtight linking and security assurance often demanded by mission
critical operations.

Further ahead on Replicant roadmap is the task of adding minor syntax extensions
and/or usage conventions for working with special "fat" primitives -- built-in
vectors of primitives (with variable dimensionality) which are implicitly
unsynchronized (think Python arrays, but with added dimensionality and built-in
usage extensions).  We believe this feature to be of great interest in
scientific computing domains and it is non-coincidentally an analog of the
design pattern associated with how a multi-process computation performs high
volume IPC (i.e., via a large interprocess shared buffer).

Project Background
------------------

Python originated in an earlier era of computing wherein the longer term implications of certain design choices (e.g., the presence of GIL, a refcount-based objects, etc.) were not apparent. As a result, for example, it is common for C extension modules to use static variables to store module-global values and data structures, making them inherently global in nature (and therefore difficult to multiply instantiate in a hypothetical environment where independent interpreter contexts are the norm).

SoundSpectrum, the company funding Replicant development, has been authoring and selling real-time music visualization software for over ten years.  Although SoundSpectrum focuses on retail sales, its software has been licensed for use by Apple, Microsoft, and various live production tours.  Starting with the Aeon visualizer, SoundSpectrum began to embedding Python heavily to facilitate visual content creation.  SoundSpectrum's C/C++ codebase is over 15 years in its evolution and is best characterized as a codebase you'd expect to see in a large video game production house.  SoundSpectrum's codebase implements a large performance-oriented class set for both Windows and OS X, using major APIs such as Direct3D, OpenGL, Cocoa/CF, and Win32.

In the course of content creation in Aeon, Python runtime performance became an increasing concern, and SoundSpectrum's engineering leadership developed an increasing interest in using independently executing interpreters in other threads to improve performance and remove real-time latency issues.  In particular, simple arithmetic in tight loops would adversely affect performance enough to be noticeable in real-time, and competition for the GIL limited multi-core performance of fundamentally independent functionality that in principle should execute in separate interpreter instances without interference.  After much back and forth on the Python listserv, Andy O'Meara set out to actively explore new implementation possibilities.

Design Philosophy
-----------------

Python's dynamic nature imposes significant challenges towards any Python implementation, and Replicant is no exception.  Our approach is therefore to regard rare or uncommon language usage cases as something that would place Replicant in a compatible (but potentially lower performing) mode that offers the support necessary for the language feature being invoked. To choose a small representative example, certain operations (e.g. exec()) may force a dictionary of locals to be created on demand, which may not have been needed otherwise during vanilla execution due to how the code was compiled).  

Meanwhile, the LLVM ecosystem seems to becoming the de facto industry standard compiler.  This is consistent with news events such as FreeBSD's recent deprecation of GCC in favor of LLVM/clang and the fact that Apple has used LLVM as its default compiler technology since 2010.  As the new compiler standard, LLVM receives enterprise-level attention and support from the largest software and hardware companies, meaning that it will continue to make large strides forward in every layer of its components for years and decades to come.  So as LLVM grows and benefits from the enterprises that make up its ecosystem, Replicant will also receive those benefits.


Operation Overview
------------------

    foo.py   ---------------+
                            |
                            v
                ------------------------
               |   Replicant Compiler   |
               |            v           |
               |           AST          |
               |            v           |
               |      Processed AST     |<===  Calls into Replicant C/C++ library 
               |            v           |      for RObject manipulation, GC interaction
               |       LLVM Bytecode    |   
                ------------------------
                            |
                            +--->  compilation to native executable
                            |
    JIT-based execution <---+
                       
    // Loading and executing Replicant-compiled LLVM bytecode at runtime
    void MyClass::DoFoo( RContext* context ) 
    {
    	robject_t* module = context -> LoadModuleBC( "/path/foo.bc" );
        // ...
        module -> Run();
        // ...
    }

When a Python script is run under Replicant (or when the embedding API module is used to run a script), the following chain of events occurs:

0. foo.py is lexed and parsed into an Abstract Syntax Tree (AST)

1. The AST undergoes a series of transformations which perform some lightweight semantic analysis and progressively refine or "lower" the AST.

2. Python objects and language constructs are transformed into calls into the Replicant runtime layer; e.g., an attribute query in the Python source is converted to a "get attribute" runtime call.

3. After the AST is sufficiently "flat", the LLVM code generator turns it into an LLVM bytecode fragment which is suitable for execution downstream (either via native code generation or via the LLVM JIT).

4. If certain runtime services are not yet available (e.g., the Replicant GC subsystem), they are started.

4. An Replicant execution context (RContext) is created/obtained.

6. The RContext is given the code fragment for execution (in which case calls back into the embedded app may occur).

7. Using the high-level RObject C/C++ support layer, explicit calls into the code fragment can now also occur (which can make calls back into the embedding app).

8. When the code fragment is no longer referenced (i.e., when the owning RContext is destroyed), it may be released.

 
About the Authors
-----------------

**Andy O'Meara** is CTO of [SoundSpectrum](http://soundspectrum.com), a music visualization software company founded in 2000, best known for the music visualizers G-Force, WhiteCap, and Aeon. These products are sold retail and have also been licensed for use by Apple, Microsoft, and various concert production tours. He specializes in realtime high-performance computation and graphics and first began to conceptualize the need for an alternative Python implementation in 2009. Andy believes in a next-generation Python implementation that can reap the benefits of unrestricted multi-threading, type inference and analysis, LLVM, and fully independent interpreter contexts.

**Joel Stanley** is a staff software designer at SoundSpectrum, Inc. He has an academic and professional background in compilers, programming languages, simulation frameworks, formal methods, and high-assurance software. Some of his past projects have used LLVM extensively. After many years of doing professional Haskell development, he became interested in Python and dynamic language compilation, and is now involved in building innovative Python tools for a multi-core world. 


Why "Replicant"?
----------------

A "replicant" is a term in Ridley Scott's 1982 film *Blade Runner* that refers to an artificially created human being.  In the film, replicants are endowed with most or all the qualities and characteristics that most people today would associate with the words "human" and "sentient", so much so that some of their faculties were also designed to exceed the level of their creators.  Although replicants were created to perform jobs and roles that most would be unwilling or unable to do, they were treated as no more than machines and property, requiring "retirement" once they were obsolete or became too self-aware.  Replicants resisting retirement were met with lethal hostility, forcing some of them to reflect on justice and their role in the universe -- in their own tragic, individualized, and constrained way.


Questions?
----------

Want to know more about Replicant or contribute?  Please e-mail replicant@soundspectrum.com. 


![Replicant Logo](https://raw.github.com/soundspectrum/replicant/master/replicant-logo.png) 

*It's too bad she won't live.  Then again, who does.*

