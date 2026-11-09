---
name: short-read-alignment-to-reference-catalog
description: Use when you need to quantify cross-mapping between a query sequence
  set (e.g., fungal marker genes) and a reference catalog, or validate taxonomic database
  specificity via simulated read mapping.
when_to_use_negative:
- Query and reference are proteins/translated sequences — use Diamond blastx with
  coverage >95%, identity >99%, e-value <10e-10 thresholds instead
- Input is a functional profiling task where reads should be annotated against FunOMIC-P
  rather than tested for cross-mapping against a bacterial reference catalog
- The query sequence set is small enough to align directly without ART read simulation
  (simulation was only needed because of memory limits)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: ART
  role: Simulates paired-end Illumina reads covering all query marker-gene sequences
    at ~1-fold coverage when direct alignment exceeds memory limits
- name: bowtie2
  role: Aligns the simulated (or real) short reads to the reference catalog index
    (e.g., UHGG)
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: Samtools v1.9
  role: Filters bowtie2 alignments, keeping mapped reads with >80% of read length
    aligned and q-score over 30
  repo: http://www.htslib.org/
- name: FunOMIC
  role: Pipeline whose FunOMIC-T marker-gene database is the query set and whose workflow
    wraps the mapping step
  repo: https://github.com/ManichanhLab/FunOMIC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:10c59c9fb518a6379355aa55e8c47ab5bf218646b278820707194b2ed815ff55
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - ART
  - FunOMIC
  - Samtools
  - Samtools v1.9
  - bowtie2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# short-read-alignment-to-reference-catalog

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns short sequencing reads (real or simulated) against a large reference catalog (e.g., the UHGG catalog of 204,938 non-redundant gut prokaryote genomes or the FunOMIC-T fungal marker-gene database) with bowtie2, then applies stringent Samtools-based mapping filters to quantify cross-mapping/contamination rates.

## When to use

Use when you need to quantify cross-mapping between a query sequence set (e.g., fungal marker genes) and a reference catalog, or validate taxonomic database specificity via simulated read mapping.

## When NOT to use

- Query and reference are proteins/translated sequences — use Diamond blastx with coverage >95%, identity >99%, e-value <10e-10 thresholds instead
- Input is a functional profiling task where reads should be annotated against FunOMIC-P rather than tested for cross-mapping against a bacterial reference catalog
- The query sequence set is small enough to align directly without ART read simulation (simulation was only needed because of memory limits)

## Inputs

- Query sequence set, e.g., 1.69 million fungal single-copy marker genes (FASTA)
- Reference catalog, e.g., UHGG catalog (204,938 non-redundant genomes from 4,644 gut prokaryotes)
- Optionally, simulated paired-end Illumina reads generated with ART (~22 million read pairs, 1-fold coverage)

## Outputs

- Bowtie2 alignment SAM/BAM file of query reads against the reference catalog
- Filtered mapped-read set (>80% aligned length, q-score >30)
- Mapping-rate summary table and figure quantifying contamination (e.g., <0.01% fungal marker reads mapping to UHGG)

## How to apply

Build a bowtie2 index of the reference catalog (e.g., UHGG) and, when the query set is too large to align directly (1.69 million FunOMIC-T marker genes exceeded memory limits), simulate paired-end Illumina reads covering the query sequences with ART at ~1-fold coverage (~22 million read pairs). Align the reads to the reference with bowtie2 and process the SAM output with Samtools v1.9. Keep only mapped reads where more than 80% of the read length aligned with a q-score over 30 — an exceedingly strict threshold chosen to minimize spurious cross-kingdom hits. Finally, compute the fraction of query-derived reads that map to the reference and report it as a mapping-rate table/figure; acceptance here is <0.01% of fungal marker-gene reads mapping to UHGG.

## Related tools

- **ART** (Simulates paired-end Illumina reads covering all query marker-gene sequences at ~1-fold coverage when direct alignment exceeds memory limits)
- **bowtie2** (Aligns the simulated (or real) short reads to the reference catalog index (e.g., UHGG)) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **Samtools v1.9** (Filters bowtie2 alignments, keeping mapped reads with >80% of read length aligned and q-score over 30) — http://www.htslib.org/
- **FunOMIC** (Pipeline whose FunOMIC-T marker-gene database is the query set and whose workflow wraps the mapping step) — https://github.com/ManichanhLab/FunOMIC

## Examples

```
FunOMIC.sh -1 read1.fastq -2 read2.fastq -p output_prefix -o output_folder -a bacterialDB_path -b FunOMIC-T_folder -c FunOMIC-P_folder -t #_threads
```

## Evaluation signals

- Mapping rate of fungal marker-gene reads to UHGG is below 0.01% after the >80% aligned-length / q>30 filter
- Filter logic keeps only reads where more than 80% of read length aligned with a q-score over 30 (verify with samtools view flags/MQ and alignment-length checks)
- Simulated read set covers the full query set at ~1-fold coverage (e.g., ~22 million ART read pairs, ~11.2 GB) with no dropped markers
- Summary table/figure reports the mapping-rate fraction and matches the reported <0.01% value
- Reverse-direction check: bacterial mock reads (903 genomes / 458 species) map to FunOMIC-T at <0.01%, confirming symmetric specificity

## Limitations

- Read simulation with ART (rather than direct alignment) was required because of memory limits, adding a simulation-based approximation to the contamination estimate
- High inter-kingdom conservation of protein-coding genes means some bacterial cross-mapping is not totally preventable, even with strict mapping thresholds
- Coverage and specificity conclusions depend on the completeness of the fungal genomes underlying FunOMIC-T (only genomes with ≥30 single-copy marker genes were retained)

## Evidence
