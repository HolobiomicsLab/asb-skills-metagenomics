---
name: organism-taxonomic-accuracy-assessment
description: Use when you need to quantify how accurately a metatranscriptome annotation
  pipeline recovers known organism identities (species/genus) and functions from reads
  with simulated ground-truth labels, e.g. to compare pipeline versions like SAMSA2
  vs SAMSA 1.0.
when_to_use_negative:
- Input reads are real experimental data with no simulated ground-truth labels — annotation-to-truth
  matching cannot be performed
- You need quantitative differential abundance analysis between control and experimental
  samples rather than annotation-accuracy benchmarking
- The pipeline of interest depends on a web service (e.g. MG-RAST) whose databases
  are not under user control, preventing a reproducible local-database comparison
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: SAMSA2
  role: Pipeline providing the annotation and aggregation scripts (DIAMOND_analysis_counter.py)
    and setup_and_test sample data used to run and score the simulated reads
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Accelerated BLAST-like aligner used to annotate reads against local, DIAMOND-indexed
    RefSeq and SEED Subsystems databases (version 0.8.3)
  repo: https://github.com/bbuchfink/diamond
- name: MetaSim
  role: Simulator used to generate the gut microbiome metatranscriptome reads with
    known ground-truth organism and function labels
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:f98c19cd839762dfa8c20025c3f5b3da9cdaa8607f7edf9f1256e12085061648
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DIAMOND
  - MetaSim
  - SAMSA2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# organism-taxonomic-accuracy-assessment

## Summary

Benchmark organism-level annotation accuracy of a metatranscriptome pipeline by annotating MetaSim-simulated gut microbiome reads against local RefSeq/SEED databases with DIAMOND (SAMSA2) and matching each read's annotation back to its simulated ground-truth label at species, genus, and SEED Subsystems hierarchy levels.

## When to use

Use when you need to quantify how accurately a metatranscriptome annotation pipeline recovers known organism identities (species/genus) and functions from reads with simulated ground-truth labels, e.g. to compare pipeline versions like SAMSA2 vs SAMSA 1.0.

## When NOT to use

- Input reads are real experimental data with no simulated ground-truth labels — annotation-to-truth matching cannot be performed
- You need quantitative differential abundance analysis between control and experimental samples rather than annotation-accuracy benchmarking
- The pipeline of interest depends on a web service (e.g. MG-RAST) whose databases are not under user control, preventing a reproducible local-database comparison

## Inputs

- MetaSim-simulated gut microbiome metatranscriptome reads in FASTQ form with known ground-truth organism and function labels (SAMSA2 setup_and_test sample data)
- Local, DIAMOND-indexed NCBI RefSeq bacteria database for organism annotation
- Local, DIAMOND-indexed SEED Subsystems database for hierarchical functional ontology

## Outputs

- Per-read DIAMOND annotation results per simulated metatranscriptome file
- Ranked abundance summaries produced by DIAMOND_analysis_counter.py
- accuracy_metrics.csv with species-level and genus-level organism accuracy and exact vs hierarchy-level functional accuracy

## How to apply

Obtain simulated metatranscriptome reads with known ground-truth organism and function labels (e.g. the MetaSim-simulated gut microbiome sample data in the SAMSA2 repository's setup_and_test folder). Annotate reads with DIAMOND version 0.8.3 against local, DIAMOND-indexed reference databases — NCBI RefSeq for organisms and SEED Subsystems for functions — using SAMSA2's custom-database annotation option. Run DIAMOND_analysis_counter.py to produce a ranked abundance summary of the DIAMOND results from each simulated metatranscriptome file. Match each read's annotation to its simulated ground-truth label at the species, genus, and SEED Subsystems hierarchy levels, then compute species/genus accuracy and exact versus any-correct-hierarchy-level functional accuracy, writing the metrics to accuracy_metrics.csv.

## Related tools

- **SAMSA2** (Pipeline providing the annotation and aggregation scripts (DIAMOND_analysis_counter.py) and setup_and_test sample data used to run and score the simulated reads) — https://github.com/transcript/samsa2
- **DIAMOND** (Accelerated BLAST-like aligner used to annotate reads against local, DIAMOND-indexed RefSeq and SEED Subsystems databases (version 0.8.3)) — https://github.com/bbuchfink/diamond
- **MetaSim** (Simulator used to generate the gut microbiome metatranscriptome reads with known ground-truth organism and function labels)

## Examples

```
git clone https://github.com/transcript/samsa2.git; bash setup_and_test/package_installation.bash; bash setup_and_test/full_database_download.bash; python DIAMOND_analysis_counter.py
```

## Evaluation signals

- Genus-level organism accuracy on the simulated metatranscriptomes should approximate the reported ~95% for SAMSA2 (vs 91% for SAMSA 1.0 on the same datasets)
- Hierarchy-level functional accuracy (any-correct SEED Subsystems level) should exceed 95%, per the paper's reported benchmark figures
- Each read in the summary must be matchable to its simulated ground-truth label at species, genus, and SEED hierarchy levels before computing accuracy_metrics.csv
- All reads must be annotated with DIAMOND 0.8.3 against local, DIAMOND-indexed databases to ensure the comparison to SAMSA 1.0 is reproducible

## Limitations

- DIAMOND is set by default to run in fast mode, which prioritizes speed; sensitive mode may improve accuracy but significantly increases memory requirements
- Accuracy figures are specific to MetaSim-simulated gut microbiome data and may not transfer to real community metatranscriptomes
- SAMSA2 requires Python 2.7 for aggregation scripts and was only tested on Linux

## Evidence

- [other] Validation: genus-level accuracy should approximate the reported ~95% (vs 91% for SAMSA 1.0): "Accuracy95% (genus level)91% (genus level)"
- [readme] SAMSA2 supports custom local databases for annotation: "Option to annotate against custom databases."
- [readme] DIAMOND_analysis_counter.py creates the ranked abundance summary used in scoring: "use "DIAMOND_analysis_counter.py" to create a ranked abundance summary of the DIAMOND results from each metatransciptome file."
- [readme] Databases must be local and DIAMOND-indexed: "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [methods] Sample data for the test/benchmark is provided in setup_and_test: "git clone https://github.com/transcript/samsa2.git"
