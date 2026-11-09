---
name: gene-count-qc-verification
description: Use when reproducing or validating gene catalog construction counts —
  e.g. after running MetaGeneMark CDS prediction or after BLASTP/Jaccard de-redundification
  — to check totals and per-source partitions against published VIRGO values.
when_to_use_negative:
- Input is already a final annotated gene catalog or abundance table (e.g. summary.Count.txt
  from a VIRGO mapping run) rather than raw prediction or clustering outputs requiring
  count validation
- The gene catalog being built is not the vaginal microbiome VIRGO-style catalog,
  so published count thresholds (~5.5M predicted, 948,158 non-redundant) are not meaningful
  invariants
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaGeneMark
  role: Predicts CDSs from metagenomic assembly contigs and isolate genome sequences
    (version 3.25, default parameters); its output totals are the primary QC checkpoint
    (~5,509,298 predicted CDSs).
- name: VIRGO
  role: The gene catalog pipeline and database whose output structure (1_VIRGO directory
    with 0.geneLength.txt) supplies the count-verification targets and annotation
    files.
  repo: github.com/ravel-lab/VIRGO
- name: Wu-blastp
  role: Performs the all-vs-all BLASTP protein comparison whose clustering results
    feed the de-redundification step being count-verified.
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:4dec825a83c5cee72900027d9e294ea75cf42588bc166418d23e7251f52970b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaGeneMark
  - VIRGO
  - Wu-blastp
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# gene-count-qc-verification

## Summary

Verification skill for quantitative checkpoints in vaginal gene-catalog construction: confirming total predicted CDS counts (~5,509,298 from MetaGeneMark) and the de-redundified non-redundant gene count (948,158 genes >99 bp, 17.2% of original predictions) with correct source-partition breakdowns.

## When to use

Use when reproducing or validating gene catalog construction counts — e.g. after running MetaGeneMark CDS prediction or after BLASTP/Jaccard de-redundification — to check totals and per-source partitions against published VIRGO values.

## When NOT to use

- Input is already a final annotated gene catalog or abundance table (e.g. summary.Count.txt from a VIRGO mapping run) rather than raw prediction or clustering outputs requiring count validation
- The gene catalog being built is not the vaginal microbiome VIRGO-style catalog, so published count thresholds (~5.5M predicted, 948,158 non-redundant) are not meaningful invariants

## Inputs

- MetaGeneMark prediction output (CDS coordinates) from assembled vaginal metagenome contigs
- MetaGeneMark prediction output from urogenital isolate genome sequence assemblies
- De-redundified non-redundant gene list from Wu-blastp all-vs-all comparison and Jaccard clustering
- Assembly-source provenance labels for each predicted CDS (metagenomic vs. isolate genome)

## Outputs

- Per-gene length annotation table (0.geneLength.txt, gene length in bp)
- Total predicted CDS count with per-source tallies (expected ~5,509,298; ~80% metagenomic / ~20% isolate)
- Non-redundant CDS count summary with source partitions (expected 948,158 total; 634,288 / 54,500 / 371,099)
- Fraction-retained statistic (expected 17.2% of original predictions)

## How to apply

After predicting CDSs on metagenomic assemblies and urogenital isolate genomes with MetaGeneMark (version 3.25, default parameters), parse the prediction output to count genes and compute per-gene lengths, attributing each CDS to its source (metagenomic assembly vs. isolate genome). Validate that the total predicted CDS count is approximately 5,509,298 with roughly 80% from metagenomic assemblies and ~20% from isolate genomes, writing the per-gene length table as 0.geneLength.txt. After pooling CDSs from all three sources (in-house metagenomes, HMP metagenomes, isolate genomes) and de-redundifying via Wu-blastp all-vs-all comparison plus Jaccard clustering, count retained non-redundant genes longer than 99 bp and partition counts. Confirm the final catalog totals 948,158 non-redundant CDSs (~0.95 million) with partition counts 634,288 / 54,500 / 371,099, representing 17.2% of the original ~5.5 million predictions.

## Related tools

- **MetaGeneMark** (Predicts CDSs from metagenomic assembly contigs and isolate genome sequences (version 3.25, default parameters); its output totals are the primary QC checkpoint (~5,509,298 predicted CDSs).)
- **VIRGO** (The gene catalog pipeline and database whose output structure (1_VIRGO directory with 0.geneLength.txt) supplies the count-verification targets and annotation files.) — github.com/ravel-lab/VIRGO
- **Wu-blastp** (Performs the all-vs-all BLASTP protein comparison whose clustering results feed the de-redundification step being count-verified.)

## Examples

```
wc -l 1_VIRGO/0.geneLength.txt
```

## Evaluation signals

- Total predicted CDS count is approximately 5,509,298 across metagenomic assemblies and isolate genomes
- Source partition of predicted CDSs is roughly 80% metagenomic assemblies and ~20% urogenital isolate genome sequences
- Non-redundant catalog contains exactly 948,158 CDSs longer than 99 bp, matching the published 0.95 million figure
- Partition counts of the non-redundant catalog are 634,288 (in-house metagenomes) / 54,500 (HMP metagenomes) / 371,099 (isolate genomes)
- Fraction of retained non-redundant genes equals 17.2% of the original ~5.5 million predictions; 0.geneLength.txt line count equals the catalog size

## Limitations

- Published counts are specific to this VIRGO construction (264 vaginal metagenomes plus 308 urogenital isolate genomes); other input sets will not match these thresholds.
- VIRGO's reference-based approach has acknowledged limitations and the database has limited inclusion of viral and fungal gene sequences (10,908 fungal and 15,965 bacteriophage genes are kept in a separate 2_fungal_phage directory), so counts refer primarily to bacterial genes.
- The README does not state a DOI or repository path for the pipeline itself, so some provenance metadata must be supplied by the user.

## Evidence

- [methods] CDS prediction via MetaGeneMark totals: [evidence span withheld — anchor status: not_found]
- [results] Non-redundant catalog count: "A total of 948,158 non-redundant CDSs longer than 99 bp were identified and retained, representing 17.2% of the original 5.5 million CDSs."
- [abstract] Catalog size in abstract: "we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes"
- [readme] README geneLength file: "***0.geneLength*** contains gene length in bp"
- [readme] README catalog description: "VIRGO is a non-redundant gene catalog for the microbial communities that inhabit the human vagina. It was built using a combination of metagenomic and urogential isolate genome sequences."
