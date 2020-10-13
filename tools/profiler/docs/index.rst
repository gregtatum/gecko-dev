Gecko Profiler
==============

The Firefox Profiler is the collection of tools used to profile Firefox. This is backed
by the Gecko Profiler, which is the primarily C++ component that instruments Gecko. It
is configurable, and supports a variety of data sources and recording modes. Primarily,
it is used as a stastical profiler, where the execution of threads that have been
registered with the profile is paused, and a sample is taken. Generally, this includes
a stackwalk with combined native stack frame, JavaScript stack frames, and custom stack
frame labels.

In addition to the sampling, the profiler can collect markers, which are collected
deterministically (as opposed to statistically, like samples). These include some
kind of text description, and optionally a payload with more information.

.. toctree::
   :maxdepth: 1

   buffer
   ci-automation
   instrumenting-java
   instrumenting-javascript
   instrumenting-rust
   jstracer
   LUL
   markers-guide
   memory
   registering-threads
   samples-stackwalking
   serialization
