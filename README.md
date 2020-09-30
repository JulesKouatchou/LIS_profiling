# LIS Profiling

## With ESMF TRACE
### Reference Document

[ESMF Profiling and Tracing](http://www.earthsystemmodeling.org/esmf_releases/last_built/ESMF_refdoc/node6.html#SECTION060130000000000000000)

### Overview

The ESMF tracing tool records detailed timing information for all phases of all ESMF components executing in an application for later analysis. Trace analysis can be used to understand what happened during a program's execution and is often used for diagnosing problems, debugging, and performance analysis.

To exercise tracing in LIS, the following two calls have been placed around specific regions of the code:

       call ESMF_TraceRegionEnter(region_name)
            Record an event in the trace for this PET indicating entry into a user-defined region with the given name.
       call ESMF_TraceRegionExit(region_name)
            Record an event in the trace for this PET indicating exit from a user-defined region with the given name.
       
       

### Compilation

Before you compile the code, set the environment variable `ESMF_TRACE` to 1:

     setenv ESMF_TRACE 1

The calls related to the profiling of sections of the code will be included in the compilation process.

### Run Time

**ESMF tracing is disabled by default.** To enable tracing, set the `ESMF_RUNTIME_TRACE` environment variable to `ON`. You do not need to recompile your code to enable tracing.

       setenv ESMF_RUNTIME_TRACE ON

The default behavior is to trace all PETs of the ESMF application. Tracing can produce an extremely large number of events depending on the total number of PETs and the length of the run. To reduce output, it is possible to restrict the PETs that produce trace output by setting the `ESMF_RUNTIME_TRACE_PETLIST` environment variable.

       # Only trace PETs 0, 20, and 35 through 39
       #setenv ESMF_RUNTIME_TRACE_PETLIST "0 20 35-39"

`phase_enter` and `phase_exit` events are automatically be recorded for all `Initialize`, `Run`, and `Finalize` phases of all Components in the application. To trace only user-instrumented regions (via the `ESMF_TraceRegionEnter()` and `ESMF_TraceRegionExit()` calls), Component-level tracing can be turned off by setting:

       setenv ESMF_RUNTIME_TRACE_COMPONENT OFF
              
Trace events are flushed to file at a regular interval. If the application crashes, some of the most recent events may not be flushed to file. To maximize the number of events appearing in the trace, an option is available to flush events to file more frequently. Because this option may have negative performance implications due to increased file I/O, it is not recommended unless needed. To turn on eager flushing use:
       
       setenv ESMF_RUNTIME_TRACE_FLUSH EAGER


### Analyzing the Profiling Outputs

After running an ESMF application with tracing enabled, a directory called `traceout` will be created in the run directory and it will contain a metadata file and an event stream file `esmf_stream_XXXX` for each PET with tracing enabled. Together these files form a valid [Common Trace Format](https://diamon.org/ctf/) (CTF) trace which may be analyzed with any of the tools listed above.

## With LIS_ftiming Module

The LIS_ftiming Module is an MPI based tool containing subroutines that allow users to measure the time to run regions of the code. 
It produces the following timing statistics: 

- The overall min/max/mean times require to run each selected region of the code.
- For each processor, the min/max/mean times require to run each selected region
- For each selected region, the execution times on all processors

These  statistics are useful for analyzing load imbalances.

### Source Code Modifications

The LIS_ftiming tool is contained in the module file `LIS_ftimingMod.F90` that resides in the `core/` directory. It consists on four main subroutines:

- `Ftiming_init`: Called once to initialize the tool
- `Ftiming_On`/`Ftiming_Off`: Called to turn on/off the time measurement at specific regions of the code.
- `Ftiming_Output`: Subroutine to gather all timing numbers and produce a timing statistics text file.

We introduced a logical variable `LIS_rc%do_ftiming` to exercise the tool at run time. The following statements were added around each region of interest:

        if (LIS_rc%do_ftiming) call Ftiming_On(bloc_name)
        if (LIS_rc%do_ftiming) call Ftiming_Off(block_name)

### Run Time Setting

To turn on the tool, we need to add the following line in the `lis.config` file:

        Profiling Tool: yes
   
### Sample Output

The timing statistics below were produced by the tool. They show the overall minimum/maximum/average times it took to to run specific sections of the code with 16x16 CPUs. There are two main blocks, `LIS_init` and `LIS_run`, and all the others fall within them.

      -----------------------------------------------------------------
           Block                       Min Time    Max Time    Avg Time
      -----------------------------------------------------------------
        LIS_init                        66.9860     69.0852     67.9283
        dom_init                        17.2476     17.3603     17.3086
        dom_quilt                        0.0028      0.1462      0.0626
        soils_init                       3.1929      3.4084      3.3168
        dom_setup                        2.0920      3.1094      2.6521
        param_init                      12.2926     12.7837     12.6170
        green_setup                      6.3572      6.6997      6.5452
        rough_setup                      0.0001      0.0002      0.0001
        emiss_setup                      0.0000      0.0000      0.0000
        alb_setup                        5.8863      6.3137      6.0716
        lai_setup                        0.0000      0.0000      0.0000
        sai_setup                        0.0000      0.0005      0.0000
        sf_init                          0.0087      0.1143      0.0797
        lsm_init                         0.0086      0.1142      0.0796
        metf_init                        0.1842      0.2972      0.2106
        DA_init                          0.0000      0.0002      0.0000
        sf_setup                        36.9808     38.6601     37.6763
        lsm_setup                       36.9808     38.6601     37.6763
        sf_readrst                       0.0000      0.0565      0.0357
        lsm_readrst                      0.0000      0.0565      0.0357
        rtm_init                         0.0000      0.0000      0.0000
        appMod_init                      0.0000      0.0002      0.0001
        LIS_run                       4440.9501   4443.0569   4442.1115
        param_dynsetup                  11.6908     13.2312     12.5341
        green_read                       6.0883      6.4517      6.2574
        emiss_read                       0.0001      0.0002      0.0001
        alb_read                         5.5073      6.0749      5.7572
        lai_read                         0.0001      0.0002      0.0001
        sai_read                         0.0001      0.0002      0.0001
        lsm_dynsetup                     0.0002      0.0006      0.0003
        soils_diag                       0.0002      0.1548      0.0810
        green_diag                       0.0002      0.0004      0.0002
        emiss_diag                       0.0001      0.0002      0.0001
        rough_diag                       0.0001      0.0002      0.0001
        lai_diag                         0.0001      0.0002      0.0001
        sai_diag                         0.0001      0.0002      0.0001
        alb_diag                         0.0001      0.3347      0.2217
        metf_get                        14.4921     30.1865     23.0301
        metf_perturb                     0.0026      1.0837      0.7558
        sf_f2t                           0.0020      0.8358      0.3696
        lsm_f2t                          0.0019      0.8352      0.3693
        sf_run                           0.0020    407.1130    247.2083
        lsm_run                          0.0018    407.1126    247.2080
        sf_perturb                       0.0003      0.0242      0.0004
        lsm_perturb                      0.0001      0.0001      0.0001
        DA_run                           0.0001      0.0001      0.0001
        DA_out                           0.0001      0.0002      0.0001
        sf_output                       39.5128   4113.0237   2897.8900
        sf_writerst                      0.0022    297.8989    247.1645
        lsm_writerst                     0.0020    297.8987    247.1642
        rtm_run                          0.0001      0.0002      0.0002
        rtm_out                          0.0001      0.0001      0.0001
        appMod_run                       0.0001      0.0001      0.0001
        appMod_out                       0.0001      0.0001      0.0001
