---
name: multi-sample-contig-pooling
description: Use when building a species-level non-redundant viral contig catalog
  from multiple shotgun viral metagenomic samples, requiring per-sample assembly followed
  by cross-sample dereplication.
when_to_use_negative:
- Input is already a dereplicated contig catalog or feature table — re-clustering
  would not add species-level resolution
- Samples contain isolated viruses rather than metagenomic communities — ViroProfiler
  is specifically designed for classifying viruses in samples with isolated viruses,
  and assembly/dereplication of mixed communities is the wrong framing
- Excessive environmental contamination is present in the metagenome sequences — contamination
  increases pipeline running time and lowers precision; isolate viral fractions before
  sequencing instead
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: metaSPAdes
  role: Per-sample metagenomic assembly of quality-filtered reads in metagenomic mode
    (--meta) to produce sample contigs
  repo: https://github.com/ablab/spades
- name: CheckV
  role: Quality and completeness assessment of assembled contigs after pooling and
    length filtering
- name: MGV ani_cluster
  role: ANI clustering script used to dereplicate pooled contigs into species-level
    clusters for nrclib construction
  repo: https://github.com/snayfach/MGV
- name: ViroProfiler
  role: Containerized pipeline orchestrating assembly, pooling, dereplication, and
    downstream use of nrclib
  repo: https://github.com/deng-lab/viroprofiler
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:56dd6d5dc0a61531d0689e3d3caeacdac37b09eefe3d4faf2ceb7453400e1c62
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  repo_url: https://github.com/ablab/spades
  tools:
  - CheckV
  - MGV ani_cluster
  - ViroProfiler
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# multi-sample-contig-pooling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles quality-filtered reads per sample with metaSPAdes in metagenomic mode, pools contigs from all samples, and dereplicates them by ANI clustering into a non-redundant contig library (nrclib) used for downstream annotation and analysis in ViroProfiler.

## When to use

Use when building a species-level non-redundant viral contig catalog from multiple shotgun viral metagenomic samples, requiring per-sample assembly followed by cross-sample dereplication.

## When NOT to use

- Input is already a dereplicated contig catalog or feature table — re-clustering would not add species-level resolution
- Samples contain isolated viruses rather than metagenomic communities — ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses, and assembly/dereplication of mixed communities is the wrong framing
- Excessive environmental contamination is present in the metagenome sequences — contamination increases pipeline running time and lowers precision; isolate viral fractions before sequencing instead

## Inputs

- Quality-filtered shotgun metagenomic reads per sample (FASTQ)
- Per-sample assembled contigs from metaSPAdes metagenomic mode
- Pooled contig library after minimum-length filtering

## Outputs

- Non-redundant contig library FASTA (nrclib.fasta) with one representative per species-level cluster
- Species-level ANI/AF cluster assignments for pooled contigs
- CheckV quality/completeness assessments of assembled contigs

## How to apply

Assemble quality-filtered reads for each sample independently with metaSPAdes in metagenomic mode (ext.args = '--meta'), allocating 4 CPUs and 100 GB memory per the pipeline's default config. Merge contigs from all samples into a single pool and filter by minimum length to remove short spurious sequences, then assess contig quality and completeness with CheckV (1 CPU, 30 GB). Dereplicate the pooled contigs at the species level by clustering sequences sharing >=95% average nucleotide identity (ANI) and >=85% alignment fraction (AF), consistent with MIUViG guidelines, using the MGV ani_cluster script. Write exactly one representative contig per species-level cluster to the non-redundant contig library FASTA (nrclib.fasta), which ViroProfiler then uses for downstream annotation and analysis (taxonomy, functional annotation, host prediction, AMG identification).

## Related tools

- **metaSPAdes** (Per-sample metagenomic assembly of quality-filtered reads in metagenomic mode (--meta) to produce sample contigs) — https://github.com/ablab/spades
- **CheckV** (Quality and completeness assessment of assembled contigs after pooling and length filtering)
- **MGV ani_cluster** (ANI clustering script used to dereplicate pooled contigs into species-level clusters for nrclib construction) — https://github.com/snayfach/MGV
- **ViroProfiler** (Containerized pipeline orchestrating assembly, pooling, dereplication, and downstream use of nrclib) — https://github.com/deng-lab/viroprofiler

## Examples

```
spades.py --meta -1 sample_R1.fq.gz -2 sample_R2.fq.gz -o sample_assembly -t 4 -m 100
```

## Evaluation signals

- nrclib.fasta exists and parses as a valid FASTA file
- nrclib contains exactly one representative sequence per species-level cluster with no duplicates at the 95% ANI / 85% AF threshold
- Every cluster member shares >=95% ANI and >=85% alignment fraction with its representative contig
- All input samples contribute contigs that were assembled with metaSPAdes --meta and passed the minimum-length filter before clustering
- Representatives in nrclib pass CheckV quality assessment for viral contig quality and completeness

## Limitations

- Excessive environmental contaminations, usually found in metagenome sequences, could increase the running time of the pipeline and result in lower precision; viral fractions should be isolated before sequencing
- Dereplication thresholds (95% ANI / 85% AF) approximate viral species but cluster boundaries may not capture true viral species diversity given limited bioinformatics standards for viral communities
- Running time and resource demands are substantial (e.g., 4 CPUs / 100 GB memory per assembly, 30 GB for CheckV)

## Evidence

- [supplementary] MIUViG-aligned ANI dereplication threshold from the workflow card: [evidence span withheld — anchor status: not_found]
- [readme] MGV viral detection pipeline description: [evidence span withheld — anchor status: external_doc]
- [supplementary] nrclib downstream use finding: [evidence span withheld — anchor status: not_found]
