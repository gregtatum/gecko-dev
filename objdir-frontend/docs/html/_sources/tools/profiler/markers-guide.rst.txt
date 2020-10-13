Markers
=======

Where to Define New Marker Types
--------------------------------

The first step is to determine the location of the file, based on its dependency for XUL.
If there is no dependency, then it can be defined in the Base Profiler, which can be used
in the most locations in the codebase:

`mozglue/baseprofiler/public/BaseProfilerMarkerTypes.h
<https://searchfox.org/mozilla-central/source/mozglue/baseprofiler/public/BaseProfilerMarkerTypes.h>`__

However, if there is a XUL dependency, then it needs to be defined in the Gecko Profiler.

`tools/profiler/public/ProfilerMarkerTypes.h
<https://searchfox.org/mozilla-central/source/tools/profiler/public/ProfilerMarkerTypes.h>`__

How to Define New Marker Types
------------------------------

.. code-block:: c++

    struct YourMarker {
      static constexpr Span<const char> MarkerTypeName() {
        return MakeStringSpan("YourMarker");
      }
      static MarkerSchemaWriter MarkerTypeDisplay() {
        MarkerSchemaWriter schema;
        using S = MarkerSchemaWriter;

        // Determine where this marker shows up in the profiler.firefox.com UI.
        schema.AddLocations(S::Location::markerChart, S::Location::markerTable);

        // The labels will show up in tooltips with proper formatting of the values.
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
      static void StreamJSONMarkerData(JSONWriter& aWriter,
                                        const ProfilerString8View& aString,
                                        const int64_t aBytes
                                        const ProfilerString8View& aURL) {
        // This JSONWriter will write out the custom information for this
        // marker payload. The arguments are populated by the variadic
        // profiler_add_marker<T> function call.
        aWriter.StringProperty("myString", aString);
        aWriter.IntProperty("myBytes", aBytes);
        aWriter.StringProperty("aURL", aurl);
      }
    };

Then in your code, this marker can be used like so:

.. code-block:: c++

    #include "GeckoProfiler.h"

    profiler_add_marker<YourMarker>({
        "YourMarker",
        // The default marker options, see documentation below.
        geckoprofiler::category::DOM.WithOptions(
            MarkerTiming::Instant(),
            MarkerInnerWindowId(innerWindowId)),
        // The rest of the arguments are variadic, and should match the order
        // in StreamJSONMarkerData.
        myString, myBytes, myUrl);


The full list of available categories is located in:

`mozglue/baseprofiler/public/ProfilingCategoryList.h
<https://searchfox.org/mozilla-central/source/mozglue/baseprofiler/public/ProfilingCategoryList.h>`__

Marker Architecture Description
-------------------------------

The above sections should give all the information needed for adding your own marker
types. However, if you are wanting to work on the marker architecture itself, this
section will describe how the system works.

TODO:
 * Briefly describe the buffer and serialization.
 * Describe the template strategy for generating marker types
 * Describe the serialization, and link to profiler front-end docs on marker processing (if they exist)
