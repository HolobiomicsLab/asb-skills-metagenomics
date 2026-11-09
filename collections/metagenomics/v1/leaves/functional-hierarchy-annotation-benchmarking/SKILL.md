---
name: functional-hierarchy-annotation-benchmarking
description: Use when evaluating or comparing annotation accuracy of metatranscriptome
  pipelines on simulated data with known ground-truth organism and function labels,
  e.g. reproducing SAMSA2's ~95% genus-level accuracy benchmark vs SAMSA 1.0.
when_to_use_negative:
- Input is real (non-simulated) metatranscriptome data with no ground-truth labels
  — accuracy cannot be computed without simulated truth
- The goal is differential abundance analysis between control and experimental samples
  (use SAMSA2's DESeq step instead)
- Only web-accessible databases are available and no local DIAMOND-indexed database
  can be built
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: SAMSA2
  role: Pipeline providing custom-database annotation, DIAMOND_analysis_counter.py
    aggregation, and setup_and_test sample data with simulated reads
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Accelerated BLAST-like aligner (version 0.8.3) used to annotate simulated
    reads against local DIAMOND-indexed RefSeq and SEED Subsystems databases
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:a13eb60b407fddfadeace29a43d96bdbf06cdd00c4ac104bca291f3eebfb4d6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DIAMOND
  - SAMSA2
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# functional-hierarchy-annotation-benchmarking

## Summary

A benchmarking protocol that quantifies organism- and functional-annotation accuracy of a metatranscriptome pipeline by annotating MetaSim-simulated reads with known ground truth (DIAMOND against local RefSeq and SEED Subsystems databases) and matching predicted annotations to simulated labels at species, genus, and SEED hierarchy levels.

## When to use

Use when evaluating or comparing annotation accuracy of metatranscriptome pipelines on simulated data with known ground-truth organism and function labels, e.g. reproducing SAMSA2's ~95% genus-level accuracy benchmark vs SAMSA 1.0.

## When NOT to use

- Input is real (non-simulated) metatranscriptome data with no ground-truth labels — accuracy cannot be computed without simulated truth
- The goal is differential abundance analysis between control and experimental samples (use SAMSA2's DESeq step instead)
- Only web-accessible databases are available and no local DIAMOND-indexed database can be built

## Inputs

- MetaSim-simulated gut-microbiome metatranscriptome reads (FASTQ) with known ground-truth organism labels
- Simulated ground-truth functional labels (SEED Subsystems hierarchy)
- DIAMOND-indexed local NCBI RefSeq bacteria database
- DIAMOND-indexed local SEED Subsystems database
- SAMSA2 repository sample data (setup_and_test)

## Outputs

- DIAMOND per-read annotation results for each simulated metatranscriptome file
- Ranked abundance summaries from DIAMOND_analysis_counter.py
- accuracy_metrics.csv containing species-level, genus-level, exact SEED function, and any-correct-hierarchy-level functional accuracy

## How to apply

Obtain MetaSim-simulated gut-microbiome metatranscriptome reads plus their simulated ground-truth organism and function labels from the SAMSA2 repository sample data (setup_and_test). Annotate reads with DIAMOND (version 0.8.3) against local, DIAMOND-indexed reference databases — NCBI RefSeq for organisms and SEED Subsystems for hierarchical functional ontology — using SAMSA2's custom-database annotation option. Run DIAMOND_analysis_counter.py to create a ranked abundance summary of the DIAMOND results from each simulated metatranscriptome file. Match each read's annotation back to its simulated ground-truth label at species, genus, and SEED Subsystems hierarchy levels, then compute organism-level accuracy (species and genus) and functional accuracy (exact SEED function match and any-correct-hierarchy-level match), writing metrics to accuracy_metrics.csv. The rationale is that local, user-controlled databases make the benchmark reproducible, unlike web-service-dependent SAMSA 1.0 via MG-RAST.

## Related tools

- **SAMSA2** (Pipeline providing custom-database annotation, DIAMOND_analysis_counter.py aggregation, and setup_and_test sample data with simulated reads) — https://github.com/transcript/samsa2
- **DIAMOND** (Accelerated BLAST-like aligner (version 0.8.3) used to annotate simulated reads against local DIAMOND-indexed RefSeq and SEED Subsystems databases) — https://github.com/bbuchfink/diamond

## Examples

```
python DIAMOND_analysis_counter.py simulated_mts_diamond.txt simulated_summary.txt
```

## Evaluation signals

- Genus-level organism accuracy on simulated reads should approximate the reported ~95% (vs 91% for SAMSA 1.0)
- Hierarchy-level functional accuracy (any correct SEED Subsystems hierarchy level) should exceed 95%, per the paper's benchmark figures
- Every simulated read's annotation is matched against its ground-truth label at species, genus, and SEED hierarchy levels with no unmatched reads
- accuracy_metrics.csv is produced with all four metrics populated (species, genus, exact SEED function, hierarchy-level match)
- Ranked abundance summaries from DIAMOND_analysis_counter.py are generated for every simulated metatranscriptome input file

## Limitations

- Benchmarks depend on MetaSim-simulated data; accuracy on real communities may differ since simulation simplifies community structure
- DIAMOND runs in fast mode by default, prioritizing speed over accuracy; sensitive mode may change accuracy but significantly increases memory requirements
- SAMSA2 requires Python 2.7 for aggregation scripts (tested on Linux only); Python 3 support is incomplete
- Full reference databases must be downloaded and DIAMOND-indexed locally, which is storage- and setup-intensive

## Evidence

- [discussion] SAMSA2 achieves 95% genus-level accuracy vs 91% for SAMSA 1.0: "Accuracy95% (genus level)91% (genus level)"
- [methods] DIAMOND annotation against local indexed databases: "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [methods] Aggregation step creates ranked abundance summaries: "use "DIAMOND_analysis_counter.py" to create a ranked abundance summary of the DIAMOND results from each metatransciptome file"
- [discussion] SEED Subsystems used for hierarchical functional ontology: "and SEED Subsystems for hierarchical ontologies of functional activity groups"
- [readme] Full databases downloaded via setup_and_test script: [evidence span withheld — anchor status: external_doc]
