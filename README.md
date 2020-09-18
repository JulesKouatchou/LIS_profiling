# LIS Profiling

## Overview



## Compilation

Before you compile the code, set the environment variable `ESMF_TRACE` to 1:

     setenv ESMF_TRACE 1

The calls related to the profiling of sections of the code will be included in the compilation process.

## Run Time

Though the profiling calls are 

       setenv ESMF_RUNTIME_TRACE ON
       setenv ESMF_RUNTIME_TRACE_COMPONENT ON
       setenv ESMF_RUNTIME_PROFILE_OUTPUT SUMMARY
       
       # Only trace PETs 0, 20, and 35 through 39
       #setenv ESMF_RUNTIME_TRACE_PETLIST "0 20 35-39"
       
       # This option may have negative performance implications due to increased file I/O.
       # It is not recommended unless needed.
       #setenv ESMF_RUNTIME_TRACE_FLUSH EAGER


## Analyzing the Profiling Outputs


