---
name: simulated-read-ground-truth-comparison
description: 'Use when validating a metatranscriptome annotator''s accuracy: you have
  simulated reads (e.g. MetaSim gut microbiome) with known ground-truth organism and
  SEED Subsystems labels and need to quantify genus-level and functional annotation
  accuracy.'
when_to_use_negative:
- The data are real (non-simulated) metatranscriptomes with no ground-truth labels
  — accuracy cannot be computed and this becomes an abundance/differential analysis
  task instead
- The input is already a ranked abundance or feature table; this skill operates on
  raw per-read annotation output, not aggregated tables
- The goal is speed/runtime benchmarking rather than annotation-accuracy measurement
  — the paper benchmarks runtime separately from the accuracy comparison
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: SAMSA2
  role: Pipeline under test; its setup_and_test folder supplies simulated sample data
    and its aggregation scripts (DIAMOND_analysis_counter.py) summarize per-read annotations
    for accuracy scoring
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Accelerated BLAST-like aligner (version 0.8.3 in the benchmark) used to annotate
    simulated reads against local, DIAMOND-indexed RefSeq and SEED Subsystems databases
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:81cd989312b3793f7e2a362992967144526cae5a94f14578714c76a6c7e652f4
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# simulated-read-ground-truth-comparison

## Summary

Benchmark an annotation pipeline by annotating MetaSim-simulated metatranscriptome reads with known ground-truth organism and function labels, then computing species/genus and SEED hierarchy accuracy — as SAMSA2 did to report 95% genus-level accuracy vs 91% for SAMSA 1.0.

## When to use

Use when validating a metatranscriptome annotator's accuracy: you have simulated reads (e.g. MetaSim gut microbiome) with known ground-truth organism and SEED Subsystems labels and need to quantify genus-level and functional annotation accuracy.

## When NOT to use

- The data are real (non-simulated) metatranscriptomes with no ground-truth labels — accuracy cannot be computed and this becomes an abundance/differential analysis task instead
- The input is already a ranked abundance or feature table; this skill operates on raw per-read annotation output, not aggregated tables
- The goal is speed/runtime benchmarking rather than annotation-accuracy measurement — the paper benchmarks runtime separately from the accuracy comparison

## Inputs

- MetaSim-simulated gut-microbiome metatranscriptome reads in FASTQ form (or SAMSA2 repository sample data from setup_and_test)
- Ground-truth labels per simulated read: organism (species/genus) and SEED Subsystems function
- Local DIAMOND-indexed reference databases: NCBI RefSeq (organisms) and SEED Subsystems (hierarchical functional ontology)

## Outputs

- Ranked abundance summaries of DIAMOND annotations per simulated file (via DIAMOND_analysis_counter.py)
- accuracy_metrics.csv with organism-level accuracy (species, genus) and functional accuracy (exact SEED function, any-correct-hierarchy-level)
- Benchmark comparison figures, e.g. ~95% (SAMSA2) vs 91% (SAMSA 1.0) genus-level accuracy

## How to apply

Obtain the simulated metatranscriptome reads and their known ground-truth labels (SAMSA2 provides these in its setup_and_test sample data). Annotate each read with DIAMOND (version 0.8.3 in the SAMSA2 benchmark) against a local, DIAMOND-indexed database such as NCBI RefSeq for organisms and SEED Subsystems for functions — SAMSA2's custom-database option is what makes local, reproducible annotation possible. Run DIAMOND_analysis_counter.py to condense per-read DIAMOND line-item results into a ranked abundance summary for each file. Match each read's annotation back to its simulated ground-truth label at species, genus, and SEED Subsystems hierarchy levels, then compute organism-level accuracy (exact species and genus matches) and functional accuracy (exact SEED function match and any-correct-hierarchy-level match), writing the metrics to accuracy_metrics.csv. Validate against the paper's benchmark: approximately 95% genus-level accuracy for SAMSA2 versus 91% for SAMSA 1.0, and hierarchy-level functional accuracy exceeding 95%.

## Related tools

- **SAMSA2** (Pipeline under test; its setup_and_test folder supplies simulated sample data and its aggregation scripts (DIAMOND_analysis_counter.py) summarize per-read annotations for accuracy scoring) — https://github.com/transcript/samsa2
- **DIAMOND** (Accelerated BLAST-like aligner (version 0.8.3 in the benchmark) used to annotate simulated reads against local, DIAMOND-indexed RefSeq and SEED Subsystems databases) — https://github.com/bbuchfink/diamond

## Examples

```
git clone https://github.com/transcript/samsa2.git && bash setup_and_test/full_database_download.bash && python DIAMOND_analysis_counter.py annotated_reads.m8
```

## Evaluation signals

- Genus-level organism accuracy on the simulated gut microbiome reads approximates the reported ~95% (vs 91% for SAMSA 1.0)
- Hierarchy-level (any-correct-SEED-level) functional accuracy exceeds 95%, matching the paper's benchmark figures
- Every annotated read in the ranked abundance summary can be traced back to a ground-truth label (no unaccounted reads between DIAMOND output and accuracy_metrics.csv)
- Metrics file accuracy_metrics.csv contains both organism-level (species, genus) and functional (exact SEED, any-hierarchy-level) columns populated for all simulated files
- DIAMOND version matches the benchmark configuration (0.8.3) and runs in fast mode by default, or sensitive mode is explicitly noted if changed

## Limitations

- Accuracy was measured on MetaSim-simulated gut microbiome data only; real-community accuracy may differ
- DIAMOND is set by default to run in fast mode, prioritizing speed; sensitive mode may significantly increase memory requirements
- SAMSA2 aggregation requires Python 2.7, and the pipeline was only tested on Linux despite claiming any-OS support
- The simulated datasets are available only in the SAMSA2 repository with no accession IDs, limiting independent provenance

## Evidence

- [discussion] Accuracy95% (genus level)91% (genus level): "Accuracy95% (genus level)91% (genus level)"
- [methods] DIAMOND annotation against local indexed DBs: "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [methods] Ranked abundance summary via counter script: "use "DIAMOND_analysis_counter.py" to create a ranked abundance summary of the DIAMOND results from each metatransciptome file"
- [readme] RefSeq and SEED databases downloaded in setup: [evidence span withheld — anchor status: external_doc]
- [readme] DIAMOND per-read results condensed in aggregation: "DIAMOND returns results on a per-read basis, a bit like a ticker tape or a line item receipt. In the aggregation step, Python scripts condense these line-by-line results to create summary tables."
