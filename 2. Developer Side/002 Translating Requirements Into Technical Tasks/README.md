# 002 Translating Requirements Into Technical Tasks

## How Should One Translate Requirements into Data Architecture: Choosing Grain, Schemas, and Storage

Once the ambiguity is cleared up and I actually know what the business expects, the real fun begins: translating those abstract rules into an actual data model. This is where developers usually fall into one of two traps—either over-engineering a hyper-normalized monster schema that takes six joins just to pull a simple metric, or throwing everything into a massive, unstructured JSON blob in a data lake and calling it a day.

### Choosing the Right Grain and Modeling Pattern

The very first architectural decision is locking down the grain. If stakeholders say they want to track "commercial engagement," that phrase means nothing until I pin down whether one row equals a single field rep interaction, a monthly account summary, or a specific product prescription event.

* **Star Schema vs. Data Vault:** For most commercial reporting layers, keeping it clean with a dimensional star schema (fact and dimension tables) saves everyone a headache. If we are dealing with multi-source Master Data Management crosswalks where lineage and auditability across disparate feeds matter more than quick query speed, a Data Vault model makes sense. But let's be honest—nobody outside of data engineering wants to write queries with fifteen table joins. Keep the reporting tier digestible.
* **Decoupled Storage and Compute:** Working in environments like Databricks or Snowflake means I don't have to stress about compute crashing under heavy ad-hoc queries the way we used to on-prem. Even so, bad partitioning strategy will burn through compute credits fast. I always make sure partitions align with how the business actually queries the data—usually by date or regional market segments—rather than arbitrary technical splits.

### Hard Boundaries: Architectural Decisions to Avoid

Just like with requirement gathering, there are certain shortcuts in data modeling that look tempting at 4:00 PM on a Thursday but turn into technical debt by next sprint:

1. **Never flatten everything into a single wide table just to avoid joins.** Sure, denormalization is great for performance, but if you create a giant 200-column flat table without a clear primary key contract, downstream BI tools will start returning duplicate sums and inflated metrics. Maintain clear dimensional boundaries.
2. **Never mix raw ingestion layers with conformed golden records.** I’ve seen teams try to mutate raw source data directly in-place to fix a bad record. Always follow a multi-tier medallion architecture (Bronze, Silver, Gold). If you corrupt your raw audit trail, you have zero recourse when an auditor asks where a metric came from last quarter.
3. **Never hardcode business logic into table views without tracking dependencies.** If a calculated column relies on a complex conditional statement (like weighting prescriber tiers based on historical sales volume), that logic belongs in the transformation pipeline code—version-controlled and tested—not buried as an undocumented SQL snippet inside a BI report.

By anchoring our architecture to these boundaries, we build pipelines that don't just work today, but actually survive when the business inevitably pivots their commercial strategy next quarter.
