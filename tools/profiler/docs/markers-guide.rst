Markers
=======

Markers are packets of arbitrary data that are added to a profile by the Firefox code, usually to indicate something important happening at a point in time, or during an interval of time.

Each marker has a name, a category, some common optional information (timing, backtrace, etc.), and an optional payload of a specific type (containing arbitrary data relevant to that type).

Where to Define New Marker Types
--------------------------------

The first step is to determine the location of the marker type definition:

* If this type is only used in one function, or a component, it can be defined in a local common place relative to its use.
* For a more common type that could be used from multiple locations:

  * If there is no dependency on XUL, it can be defined in the Base Profiler, which can be used
in most locations in the codebase:
`mozglue/baseprofiler/public/BaseProfilerMarkerTypes.h
<https://searchfox.org/mozilla-central/source/mozglue/baseprofiler/public/BaseProfilerMarkerTypes.h>`__
  * However, if there is a XUL dependency, then it needs to be defined in the Gecko Profiler:
`tools/profiler/public/ProfilerMarkerTypes.h
<https://searchfox.org/mozilla-central/source/tools/profiler/public/ProfilerMarkerTypes.h>`__

How to Define New Marker Types
------------------------------

.. code-block:: c++

    struct YourMarker {
      static constexpr Span<const char> MarkerTypeName() {
        // This marker name must be unique in the Firefox code.
        return MakeStringSpan("YourMarker");
      }

      static void StreamJSONMarkerData(JSONWriter& aWriter,
                                        const ProfilerString8View& aString,
                                        const int64_t aBytes,
                                        const ProfilerString8View& aURL) {
        // This JSONWriter will write out the custom information for this
        // marker payload. The arguments are populated by the variadic
        // profiler_add_marker function call and related macros like PROFILER_MARKER.
        aWriter.StringProperty("myString", aString);
        aWriter.IntProperty("myBytes", aBytes);
        aWriter.StringProperty("aURL", aURL);
      }

      static MarkerSchemar MarkerTypeDisplay() {
        using MS = MarkerSchema;
        // Start by constructing a schema writer.
        // Arguments determine where this marker shows up in the profiler.firefox.com UI.
        MS schema(S::Location::markerChart, S::Location::markerTable);

        // Some labels can be specified, to display certain information in different locations.
        schema.SetChartLabel("{marker.name} {marker.data.myBytes}B");

        // These main labels will show up in tooltips with proper formatting of the values.
        // In addition, this will help not leak sensitive user information, as information
        // such as URLs can properly be sanitized.
        schema.AddKeyLabelFormat(
            "myString", "My String", S::Format::string);
        schema.AddKeyLabelFormat(
            "myBytes", "My Bytes", S::Format::bytes);
        schema.AddKeyLabelFormat(
            "myUrl", "My URL", S::Format::url);

        return schema;
      }
    };

Then in your C++ code, this marker can be used like so:

.. code-block:: c++

    #include "GeckoProfiler.h"

    PROFILER_MARKER(
        // Name, and category pair.
        "YourMarker", DOM,
        // Marker options, see documentation below.
        MarkerOptions{
            MarkerTiming::IntervalUntilNowFrom(someStartTimestamp),
            MarkerInnerWindowId(innerWindowId))},
        // The rest of the arguments are variadic, and should match the order
        // in StreamJSONMarkerData.
        myString, myBytes, myUrl);


Untyped markers are also available, when a single string is sufficient for your use-case:

.. code-block:: c++

    #include "GeckoProfiler.h"

    PROFILER_MARKER_UNTYPED(
        // Name, and category pair.
        "This happened", OTHER,
        // Marker options, may be omitted if defaults are acceptable.
        MarkerOptions{MarkerStack::Capture()});


The full list of available categories is located in:

`mozglue/baseprofiler/public/ProfilingCategoryList.h
<https://searchfox.org/mozilla-central/source/mozglue/baseprofiler/public/ProfilingCategoryList.h>`__

``MarkerSchema``, ``MarkerOptions`` and the different types of sub-options it accepts are defined in:

`mozglue/baseprofiler/public/BaseProfilerMarkersPrerequisites.h
<https://searchfox.org/mozilla-central/source/mozglue/baseprofiler/public/BaseProfilerMarkersPrerequisites.h>`__

Marker Architecture Description
-------------------------------

The above sections should give all the information needed for adding your own marker
types. However, if you are wanting to work on the marker architecture itself, this
section will describe how the system works.

TODO:
 * Briefly describe the buffer and serialization.
 * Describe the template strategy for generating marker types
 * Describe the serialization, and link to profiler front-end docs on marker processing (if they exist)
