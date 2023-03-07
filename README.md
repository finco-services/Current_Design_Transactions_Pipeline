# Prompt
_Send the candidate [this gist](https://gist.github.com/yongli-current/b79bac317764f247a5ba063af5bda721) to show them the prompt._

1. Design a system that extract Transactions and Wallet information into an appropriate data warehouse.
2. Design a pipeline for deriving user balance.
3. As the business grows, we want to build ML models for user classification, we need a pipeline for building out features that will be used to model training Features that we want to support include “direct_deposit_amount_last_30_days”, “swipes_last_30_days”, “swipe_amount_last_30_days”, “days_since_account_creation”, etc
For this 3, assume you have a lookup table ~100k entries that classifies direct deposits

```
Account name, description, deposit category
Walmart, payroll, payroll
Amazon, direct deposit, payroll
Uber, vmt transfer, gig
```


# Solution & Evaluation
We will use set of [questions and answers](https://docs.google.com/document/d/1ORnQUAsm7ZHGHYPFikkqU-n80Wo5bQaV1S1Rh8Eee_M) to determine their performance.


## Choosing Data Store 
* Has the candidate chosen a reasonable data store and explained their reasoning
* Has the candidate discussed on how to ingest the data(streaming vs batch)
* What is the schema. If TC chose a technology where the schema needs to be defined, how do we manage the schema?
* Have the candidate explain how we instrument a service (e.g. we use a metrics client)

```
Json
Avro
  - Row based storage
  - Schema included in payload
  - Ideal for ETL operations and query on all columns
  - Supports change in schema, no need additional schema versioning
Parquet 
  - column based,
  - fast for queries on subset of columns, analytics
  - need separate system for schema, DDL

```

## Validation
* How do you ensure all data is ingested and accurate
  * In the case data is missing, how do we backfill or replay data ingestion
* What are some validations we can add to the pipeline
* How do you handle duplication of data

## Data Transformation
Once candidate has proposed a system for ingesting transaction data, we'll focus on the second part which is data transformation.
* How does candidate plan on deriving customer balance.
  * Does Candidate use map-reduce or bigquery processing
* Do we foresee the analysts in our company running any of the map/reduce pipelines described above?
* What kind of language of DSL are the analysts in our company going to use in order to aggregate the data?

## Bonus: Data Governance and Security
If the candidate zooms through the exercise, then it's worth asking some follow-ups on data governance
* How do you determine data owners and stewards that ensure data standards are met 
* Data Encryption
  * At rest encryption, 
  * Limit access to prod data

