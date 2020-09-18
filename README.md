# LIS Profiling

## Reference Document

[ESMF Profiling and Tracing](http://www.earthsystemmodeling.org/esmf_releases/last_built/ESMF_refdoc/node6.html#SECTION060130000000000000000)

## Overview

The ESMF tracing tool records detailed timing information for all phases of all ESMF components executing in an application for later analysis. Trace analysis can be used to understand what happened during a program's execution and is often used for diagnosing problems, debugging, and performance analysis.

To exercise tracing in LIS, the following two calls have been placed around specific regions of the code:

       call ESMF_TraceRegionEnter(region_name)
            Record an event in the trace for this PET indicating entry into a user-defined region with the given name.
       call ESMF_TraceRegionExit(region_name)
            Record an event in the trace for this PET indicating exit from a user-defined region with the given name.
       
       

## Compilation

Before you compile the code, set the environment variable `ESMF_TRACE` to 1:

     setenv ESMF_TRACE 1

The calls related to the profiling of sections of the code will be included in the compilation process.

## Run Time

**ESMF tracing is disabled by default.** To enable tracing, set the `ESMF_RUNTIME_TRACE` environment variable to `ON`. You do not need to recompile your code to enable tracing.

       setenv ESMF_RUNTIME_TRACE ON

The default behavior is to trace all PETs of the ESMF application. Tracing can produce an extremely large number of events depending on the total number of PETs and the length of the run. To reduce output, it is possible to restrict the PETs that produce trace output by setting the `ESMF_RUNTIME_TRACE_PETLIST` environment variable.

       # Only trace PETs 0, 20, and 35 through 39
       #setenv ESMF_RUNTIME_TRACE_PETLIST "0 20 35-39"

`phase_enter` and `phase_exit` events are automatically be recorded for all `Initialize`, `Run`, and `Finalize` phases of all Components in the application. To trace only user-instrumented regions (via the `ESMF_TraceRegionEnter()` and `ESMF_TraceRegionExit()` calls), Component-level tracing can be turned off by setting:

       setenv ESMF_RUNTIME_TRACE_COMPONENT OFF
              
Trace events are flushed to file at a regular interval. If the application crashes, some of the most recent events may not be flushed to file. To maximize the number of events appearing in the trace, an option is available to flush events to file more frequently. Because this option may have negative performance implications due to increased file I/O, it is not recommended unless needed. To turn on eager flushing use:
       
       setenv ESMF_RUNTIME_TRACE_FLUSH EAGER


## Analyzing the Profiling Outputs

After running an ESMF application with tracing enabled, a directory called `traceout` will be created in the run directory and it will contain a metadata file and an event stream file `esmf_stream_XXXX` for each PET with tracing enabled. Together these files form a valid [Common Trace Format](https://diamon.org/ctf/) (CTF) trace which may be analyzed with any of the tools listed above.
