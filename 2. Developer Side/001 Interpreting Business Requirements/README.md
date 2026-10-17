# 001 Interpreting Business Requirements

Staring at a newly minted requirements document, I find myself trying to decode what stakeholders *actually* mean by phrases like "establishing a unified enterprise view." 

Does "unified" imply a deterministic exact-match pipeline, or a probabilistic crosswalk engine utilizing fuzzy matching thresholds? Too often, I catch myself immediately sketching out table schemas and PySpark jobs before fully resolving these ambiguities. We rush into implementation—spinning up Databricks clusters and writing Delta Lake ingestion scripts—while foundational business rules remain vaguely defined.

Translating high-level stakeholder objectives into concrete technical specifications requires a deliberate approach to discovery. When business users request data that is "clean and accurate," we have to push past the surface and establish quantifiable data contracts. Accurate compared to what baseline? Cleaned according to which deterministic exclusion rules? Without pinning down these edge cases early—such as how to handle null identifiers, divergent address formats, or conflicting historical timestamps—we build pipelines based on shifting sand.

Bridging the gap between vague business intent and strict architectural execution is critical for maintaining delivery momentum. Instead of treating requirement gathering as a passive handoff, we need to actively interrogate the assumptions behind every user story, trace lineage through legacy documentation, and validate edge cases upfront. That way, when we finally deploy to production, the output matches actual operational needs rather than unstated expectations.

## 002 The Developer’s Mental Framework: Questions to Ask, Questions to Avoid, and Hard Boundaries

When a ticket lands in my queue, I used to just dive straight into the code editor. Now, I try to run a quick internal sanity check before writing a single line of PySpark. We need a mental framework that bridges the gap between what the business *says* they want and what we actually have to build—without getting bogged down in endless alignment meetings.

### The Question Matrix: What I Should Ask vs. What I *Could* Ask

You can't ask stakeholders fifty questions for every minor feature request; otherwise, you become the bottleneck. I've learned to split inquiries into non-negotiable blockers versus secondary optimizations.

* **What I Should Ask (The Blockers):**
* *What defines a duplicate record here?* (If two customer entries share an email but have different phone numbers, do we merge or keep separate?)
* *What is the acceptable tolerance for data loss or latency?* (Are we talking real-time streaming validation, or is an overnight batch run fine?)
* *What exact metrics or reports depend on this specific output?* (Knowing the downstream use case changes how I model the surrogate keys.)


* **What I *Could* Ask (The Nice-to-Haves):**
* *Are there preferred naming conventions for the final reporting view?*
* *Do you anticipate adding new source feeds to this pipeline in the next two quarters?*
* *Would historical backfilling be required if we alter the transformation logic later?*

### The Red Lines: Decisions I Should *Not* Take Upon Myself

This is where developers usually get into trouble. We try to be helpful by making "reasonable assumptions," which usually backfires when the business comes back saying, "That's not what we meant at all." Here are the boundary lines I refuse to cross:

1. **Never silently drop dirty data.** If a record fails validation or has a broken foreign key, I shouldn't just write a quiet `drop()` filter in Spark and hope nobody notices. Deciding which data gets discarded is a *business* choice, not a technical convenience. If it's bad, it goes to a quarantine or dead-letter table for review.
2. **Never invent business fallback rules.** If a timestamp or primary identifier is null, I have no business hardcoding a default value like `1970-01-01` or `UNKNOWN_CUSTOMER` unless explicitly told to do so. A fake default corrupts downstream analytics faster than missing data ever would.
3. **Never alter grain without sign-off.** If the source feed provides transaction-level logs, I cannot unilaterally decide to aggregate it to a daily summary just because it makes join performance easier. Changing the grain of a table changes its entire analytical meaning.

By keeping these guardrails active, I spend less time rewriting pipelines due to miscommunication and more time actually engineering robust data systems.
