# Prompt
_Send the candidate [this gist](https://gist.github.com/flylo/4612f614e22ecb39c65e38d90f44e326) to show them the prompt._

Design a metrics framework that allows for collecting and visualizing tagged metrics at scale.

The system must include:
* A way for engineers to emit values of the metric from their services.
* A way for end-users to visualize statistics* over various time aggregations.

_*For the purpose of this exercise, the only statistics we'll collect are Min, Mean, and Max_

Example observations:
```
{
 ts: 1232,
 tags: ["service:codex","datacenter:us-east1","hostid:1234"],
 metric_name: "response_time",
 value: 123
 unit: ms
}

{
 ts: 1233,
 tags: ["service:codex","hostid:1234"],
 metric_name: "response_time",
 value: 33
 unit: ms
}
```

# Solution & Evaluation
We will use [this rubric](https://docs.google.com/spreadsheets/d/1hHn1sEmM2xFIhYrxUTvNWT9EWDA_tln8sLwGgQobuco/edit?ts=5f442c04#gid=0) to determine their performance.


## Publishing Metrics
* Have the candidate explain how we instrument a service (e.g. we use a metrics client)
* Have the candidate write out the schema that gets sent to the back-end.
* Are we sending individual observations to the back-end?
  * If so, mention that multiple services need to be instrumented, and that each service handles thousands of requests/second
  * If not, how are we aggregating and/or batching observations?
    * We should aggregate on all unique combinations of tags.
Example aggregation schema:
```
{
 ts: resolution(1232),
 tags: ["cat","dog"],
 metric_name: "response_time",
 value: 55
 statistic: "mean"
 num_observations: 20
 unit: ms
}
```

## Queuing
* Where do batches get sent? Should be some type of messaging or streaming service (Pub/Sub, Kafka, Kinesis, etc)
* What are the guarantees & assumptions of your chosen queuing technology?
  * Kinesis & Pub/Sub: At-least once
  * Kafka: exactly, at-least, or at-most once depending on how you set it up
* How to handle retries in the metrics client?
  * We need to wait for an ack from the messaging service, but that means that we'll retry if we don't get the ack. We must enforce idempotent consumption of the queue.

## Storage
* Are your writes idempotent?
* What technology are you using for persistence?
  * NoSQL: Ideal because idempotency and subsetting can be solved with a well-designed index
  * SQL: No real relational structure for the data, and no need for ACID commits. If the candidate can explain why they're using SQL it's fine.
  * Lucene-Based Search (Elasticsearch or Solr): Could be some good reasons to use this, but also some potential major downsides like indexing time query speed.
    * If the candidate chooses elasticsearch but doesn't want to actually use the native data-layer aggregation it could be a red flag.
* What is the schema of the data for storage purposes?
* How would you design the index?

Example NoSQL schema & index:
```
{
	key: "${timetamp}${metric_name}${statistic}${hash(tags))"
	value: {
		tags: ["cat", "dog"],
		metric_value: 492
		num_observations: 12
	}
}
```

## API/Service Design
* What is the client-facing schema?
* Do they expect the client to do any aggregations? If so, question their reasoning. Aggregations should most likely be done entirely on the server-side.
* Do we put inserts/upserts behind an API?
  * We could have a streaming pipeline call the API or we could have the service directly consume the queue.
* How does one aggregate a mean? min or max?
* How do we aggregate our statistics for various tags?
  * Tags should be additive, so we filter by the tags we want then aggregate the groups.

Example request:
```
{
	metric_name: "response_time",
	date_range_exclusive: [1200, 1500],
	statistic: "min",
	tags: ["dog"]
}
```

Example response:
```
{
	name: "response_time",
	units: "ms"
	statistic: "min",
	floor_timestamps: [1200, 1300, 1400],
	metric_values: [292.4, 301.2, 299.1],
}
```


## Bonus: Sample Variance & Efficient Statistics
If the candidate zooms through the exercise, then it's worth asking some follow-ups on efficient algorithms for streaming data.
* How do we aggregate the mean in our windows?
  * Answer should be something along the lines of a [streaming mean](http://www.nowozin.net/sebastian/blog/streaming-mean-and-variance-computation.html). Candidate doesn't need to know the exact formula but if they understand the concept they get major bonus points.
* Assume that end-users want to see the variance of the metric over time. How do we aggregate the variance?
	* [Streaming variance](https://www.johndcook.com/blog/standard_deviation/)
	* [Combining variances of groups](https://www.emathzone.com/tutorials/basic-statistics/combined-variance.html)
