---
name: alignment-quality-filtering
description: Use when quantifying cross-mapping or contamination between sequence
  databases (e.g., fungal markers vs. a bacterial reference) and strict alignment-quality
  criteria are needed to call a read 'mapped'.
when_to_use_negative:
- Input is already a feature/abundance table or summary counts — no alignment step
  exists to filter
- The analysis goal is functional protein annotation, where Diamond blastx thresholds
  (coverage >95%, identity >99%, e-value <10e-10) apply instead of bowtie2/samtools
  read filtering
- Sensitive detection of divergent homologs is the goal — the >80% length and q>30
  cutoffs are deliberately exceedingly strict and will discard weak-but-real cross-matches
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0091
tools:
- name: bowtie2
  role: Aligns the simulated paired-end marker-gene reads against the UHGG reference
    catalog
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: Samtools v1.9
  role: Filters bowtie2 alignments, keeping only reads with >80% aligned length and
    q-score over 30
  repo: http://www.htslib.org/
- name: ART
  role: Simulates the 22 million paired-end Illumina reads (1-fold coverage) covering
    all FunOMIC-T marker genes prior to alignment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:f9b09d1032af7636c195f6a0433cad423fb7dacf1988a5eefc2d745ad7b0d114
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
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Alignment quality filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A read-mapping readout filter that retains only high-confidence alignments — reads where more than 80% of the read length aligns to the reference with a q-score over 30 — used in FunOMIC to quantify cross-mapping of fungal marker genes against the UHGG bacterial catalog.

## When to use

Use when quantifying cross-mapping or contamination between sequence databases (e.g., fungal markers vs. a bacterial reference) and strict alignment-quality criteria are needed to call a read 'mapped'.

## When NOT to use

- Input is already a feature/abundance table or summary counts — no alignment step exists to filter
- The analysis goal is functional protein annotation, where Diamond blastx thresholds (coverage >95%, identity >99%, e-value <10e-10) apply instead of bowtie2/samtools read filtering
- Sensitive detection of divergent homologs is the goal — the >80% length and q>30 cutoffs are deliberately exceedingly strict and will discard weak-but-real cross-matches

## Inputs

- FASTQ paired-end reads simulated or sequenced from the query sequences (e.g., 22 million ART-simulated pairs covering 1.69 million FunOMIC-T marker genes)
- Reference genome catalog index for bowtie2 (e.g., the UHGG catalog of 204,938 non-redundant genomes from 4,644 gut prokaryotes)
- SAM/BAM alignment output from bowtie2

## Outputs

- Filtered BAM containing only mapped reads with >80% aligned read length and q-score >30
- Mapping-rate summary table (fraction of marker-gene reads mapping to the reference)
- Mapping-rate figure / contamination report

## How to apply

Simulate paired-end Illumina reads covering the query sequences (here, 22 million paired reads at 1-fold coverage of the 1.69 million FunOMIC-T marker genes, generated with ART to bypass memory limits) and align them to the reference catalog with bowtie2. Filter the resulting alignments with Samtools v1.9, keeping only mapped reads in which more than 80% of the read length aligned to the reference sequence with a q-score of over 30. Compute the fraction of query reads passing the filter that map to the reference; in the source workflow this produced the mapping-rate summary table and figure for the UHGG contamination check.

## Related tools

- **bowtie2** (Aligns the simulated paired-end marker-gene reads against the UHGG reference catalog) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **Samtools v1.9** (Filters bowtie2 alignments, keeping only reads with >80% aligned length and q-score over 30) — http://www.htslib.org/
- **ART** (Simulates the 22 million paired-end Illumina reads (1-fold coverage) covering all FunOMIC-T marker genes prior to alignment)

## Examples

```
bowtie2 -x uhgg -1 sim_R1.fq -2 sim_R2.fq | samtools view -bS - | samtools view -b -q 30 - > marker_vs_uhgg.filtered.bam
```

## Evaluation signals

- Acceptance criterion: fewer than 0.01% of the fungal marker-gene reads map to the UHGG after filtering, as reported in the paper
- Only reads passing both thresholds (>80% of read length aligned AND q-score over 30) are counted as mapped; verify filter logic keeps unmapped/short/low-Q reads out
- Reciprocal check: a 903-genome bacterial mock community simulated into 19,301,201 Illumina reads should show <0.01% mapping rate against FunOMIC-T (specificity 0.9999 in the ~1000:1 bacterial:fungal mixed mock)
- Samtools version recorded (v1.9) and alignment summary from bowtie2 logs should reconcile with the filtered read counts
- Output should include a mapping-rate summary table and figure consistent with the <0.01% contamination bound

## Limitations

- High inter-kingdom conservation means bacterial contamination is not totally preventable; even the strict mapping threshold cannot eliminate all cross-mapping for functional annotation
- Because of memory limits, reads must be simulated from the 1.69 million marker genes (ART, ~11.2 GB at 1-fold coverage) rather than aligning the sequences directly, which introduces simulation artifacts
- This read-filtration step addresses taxonomic mapping; no bacterial read filtration exists before functional annotation in the current FunOMIC release (proposed only for a future update)

## Evidence

- [methods] Mapped reads kept if >80% of length aligned and q-score >30: "Mapped reads were kept if more than 80 % of the length aligned to the reference sequence with a q-score of over 30"
- [methods] bowtie2 mapping of marker genes to UHGG: [evidence span withheld — anchor status: not_found]
- [results] <0.01% mapping result: "Only<0.01 % of the fungal marker genes mapped to the UHGG"
- [readme] FunOMIC requires bowtie2 and samtools: "A full installation of FunOMIC requires Python 3.3+ (modules needed: pandas, math, subprocess, argparse), R 4.0+ (packages needed: KEGGREST, RCurl), bowtie2, samtools, flash2, and diamond."
