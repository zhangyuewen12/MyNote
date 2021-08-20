## Event Time and Watermarks

Flink explicitly supports three different notions of time:

- *event time:* the time when an event occurred, as recorded by the device producing (or storing) the event
- *ingestion time:* a timestamp recorded by Flink at the moment it ingests the event
- *processing time:* the time when a specific operator in your pipeline is processing the event



For reproducible results, e.g., when computing the maximum price a stock reached during the first hour of trading on a given day, you should use event time. In this way the result won’t depend on when the calculation is performed. This kind of real-time application is sometimes performed using processing time, but then the results are determined by the events that happen to be processed during that hour, rather than the events that occurred then. Computing analytics based on processing time causes inconsistencies, and makes it difficult to re-analyze historic data or test new implementations.

### Working with Event Time

If you want to use event time, you will also need to supply a Timestamp Extractor and Watermark Generator that Flink will use to track the progress of event time. This will be covered in the section below on [Working with Watermarks](https://ci.apache.org/projects/flink/flink-docs-release-1.12/learn-flink/streaming_analytics.html#working-with-watermarks), but first we should explain what watermarks are.

