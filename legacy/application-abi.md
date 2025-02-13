WASI Application ABI
====================

In addition to the APIs defined by the various WASI modules there
are also certain expectations that the WASI runtime places on an application
that wishes to be portable across WASI implementations.

This document describes how a conforming WASI application is expected to behave
in terms of lifecycle (startup, shutdown, etc) and any exports it is expected to
include.

Current Unstable ABI
--------------------

There are two kinds of modules:

 - A *command* exports a function named `_start`, with no arguments and no return
   values.

   `_start` is the default export which is called when the user doesn't select a
   specific function to call. Commands may also export additional functions,
   (similar to "multi-call" executables), which may be explicitly selected by the
   user to run instead.

   Except as noted below, commands shall not export any mutable globals, tables,
   or linear memories.

   Command instances may assume that they will be called from the environment
   at most once. Command instances may assume that none of their exports are
   accessed outside the duration of that call.

 - All other modules are *reactors*. A reactor may export a function named
   `_initialize`, with no arguments and no return values.

   If an `_initialize` export is present, reactor instances may assume that it
   will be called by the environment at most once, and that none of their
   other exports are accessed before that call.

   After being instantiated, and after any `_initialize` function is called,
   a reactor instance remains live, and its exports may be accessed.

These kinds are mutually exclusive; implementations should report an error if
asked to instantiate a module containing exports which declare it to be of
multiple kinds.

Regardless of the kind, all modules accessing WASI APIs also export a linear
memory with the name `memory`. Data pointers in WASI API calls are relative to
this memory's index space.

Regardless of the kind, all modules accessing WASI APIs also export a table
with the name `__indirect_function_table`. Function pointers in WASI API calls
are relative to this table's index space.

When `_start` or `_initialize` is called, environments shall provide file
descriptors with indices 0, 1, and 2 representing stream resources for
standard input, standard output, and standard error. Environments may provide
additional "preopen" file descriptors that can be inspected with
`fd_prestat_get` and `fd_prestat_dir_name`. These resources may be closed at
any time.

Environments shall not access exports named `__heap_base` or `__data_end`.
Toolchains are encouraged to avoid providing these exports.

In the future, as the underlying WebAssembly platform offers more features, we
we hope to eliminate the requirement to export all of linear memory or all of
the indirect function table.

Planned Stable ABI
------------------

There is ongoing discussion about what the stable ABI might look like:

- https://github.com/WebAssembly/WASI/issues/13
- https://github.com/WebAssembly/WASI/issues/19
- https://github.com/WebAssembly/WASI/issues/24
