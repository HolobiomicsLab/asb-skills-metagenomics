---
name: metagenomic-taxonomic-classification-benchmarking
description: Use when you need to quantitatively compare taxonomic classifiers on
  metagenomic reads with known community composition, computing per-tool F-measure
  at genus and species level and runtime.
when_to_use_negative:
- Input datasets lack known ground-truth community composition, so precision/recall/F-measure
  cannot be computed against a reference
- The goal is real biological discovery on environmental samples rather than classifier
  benchmarking on controlled communities
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: RiboTaxa
  role: Pipeline under evaluation; trims reads (BBTools), filters 16S/18S reads (SortMeRNA),
    reconstructs SSU rRNA (EMIRGE + MetaRib) and classifies (QIIME2 sklearn)
  repo: https://github.com/oschakoory/RiboTaxa
- name: Kraken2
  role: Competing short-read taxonomic classifier benchmarked on the same datasets
- name: Centrifuge
  role: Competing short-read taxonomic classifier benchmarked on the same datasets
- name: SPINGO
  role: Competing species-level amplicon classifier included in the benchmark
  repo: https://github.com/GuyAllard/SPINGO
- name: SortMeRNA
  role: rRNA-specialized read filter; performed better than BBMap at extracting assemblable
    rRNA reads
- name: EMIRGE
  role: Reconstructs full-length SSU sequences from filtered reads (JOIN_THRESHOLD
    identity = 1, 40 iterations)
- name: MetaRib
  role: Combined with EMIRGE for SSU reconstruction, yielding greater population characterisation
- name: QIIME 2 Sklearn classifier
  role: Classifies reconstructed full-length SSU sequences at confidence 0.7 against
    SILVA 138
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:99079deff5af8699d303aeef53739f86434ed7b41d48a0da2ee2b78c72707719
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  repo_url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBTools
  - Centrifuge
  - EMIRGE
  - Kraken2
  - MetaRib
  - QIIME 2 Sklearn classifier
  - RiboTaxa
  - SPINGO
  - SortMeRNA
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# metagenomic-taxonomic-classification-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks SSU rRNA-based taxonomic classifiers (RiboTaxa vs Kraken2, Centrifuge, METAXA2, phyloFlash, SPINGO, BLCA, MEGAN6) on mock and synthetic human gut metagenomic communities, comparing genus- and species-level precision, recall, F-measure and wall-clock runtime.

## When to use

Use when you need to quantitatively compare taxonomic classifiers on metagenomic reads with known community composition, computing per-tool F-measure at genus and species level and runtime.

## When NOT to use

- Input datasets lack known ground-truth community composition, so precision/recall/F-measure cannot be computed against a reference
- The goal is real biological discovery on environmental samples rather than classifier benchmarking on controlled communities

## Inputs

- MOCK community metagenomic Illumina reads (paired-end FASTQ)
- Synthetic human gut community reads (HG_R1/HG_R2 FASTQ from RiboTaxa_Supp)
- Indexed SILVA SSU 138.1 database (SortMeRNA and Bowtie indices)
- QIIME2-trained SILVA 138 classifier
- Known ground-truth community composition per dataset

## Outputs

- benchmark_performance_table.csv with per-tool genus- and species-level precision, recall and F-measure (article Tables 2-3, Figure 8)
- benchmark_runtime_table.csv with wall-clock runtime per tool per dataset
- Per-tool taxonomic profile calls (genus/species assignments with relative abundances)

## How to apply

Run the RiboTaxa pipeline on each benchmark dataset: BBTools adapter/quality trimming, SortMeRNA filtering of 16S/18S reads, EMIRGE (JOIN_THRESHOLD identity = 1, 40 iterations, MIN_COV = 0.3) combined with MetaRib for full-length SSU reconstruction, and QIIME2 sklearn classification at confidence 0.7 against SILVA SSU 138.1. Run each competitor (Kraken2, Centrifuge, METAXA2, phyloFlash in BS/BE/SE modes, SPINGO, BLCA, MEGAN6) on the same inputs. Compare each tool's genus- and species-level calls to the known composition of the MOCK and synthetic human gut community, computing precision, recall and F-measure per tool per dataset, and record wall-clock runtime. The rationale for including RiboTaxa is that reconstructing full-length SSU sequences before classification avoids the high false-positive rate seen when classifying short Illumina reads directly.

## Related tools

- **RiboTaxa** (Pipeline under evaluation; trims reads (BBTools), filters 16S/18S reads (SortMeRNA), reconstructs SSU rRNA (EMIRGE + MetaRib) and classifies (QIIME2 sklearn)) — https://github.com/oschakoory/RiboTaxa
- **Kraken2** (Competing short-read taxonomic classifier benchmarked on the same datasets)
- **Centrifuge** (Competing short-read taxonomic classifier benchmarked on the same datasets)
- **SPINGO** (Competing species-level amplicon classifier included in the benchmark) — https://github.com/GuyAllard/SPINGO
- **SortMeRNA** (rRNA-specialized read filter; performed better than BBMap at extracting assemblable rRNA reads)
- **EMIRGE** (Reconstructs full-length SSU sequences from filtered reads (JOIN_THRESHOLD identity = 1, 40 iterations))
- **MetaRib** (Combined with EMIRGE for SSU reconstruction, yielding greater population characterisation)
- **QIIME 2 Sklearn classifier** (Classifies reconstructed full-length SSU sequences at confidence 0.7 against SILVA 138)

## Examples

```
git clone https://github.com/oschakoory/RiboTaxa.git; cd RiboTaxa; bash conda_virt_env.sh
```

## Evaluation signals

- RiboTaxa should reproduce F-measure of 100% at genus level and 83.7% at species level on the synthetic human gut community with no false positives
- Per-tool runtimes should fall in the reported minute-scale range (RiboTaxa ~18-26 min), with short-read classifiers (Kraken2, Centrifuge) faster
- Reconstructed sequences should be correctly assigned while direct classification of short-length Illumina reads yields a high rate of false positives
- Output tables must match the layout of article Tables 2-3 and Figure 8 (precision, recall, F-measure per tool per taxonomic level)

## Limitations

- RiboTaxa is not as fast as short-read classifiers (i.e. Kraken2, Centrifuge)
- High conservation of 16S rRNA sequence cannot discriminate all species (e.g. within Enterobacteriaceae, Lachnospiraceae); RiboTaxa may stop at genus level
- No 16S rRNA gene copy number normalization is applied, and artificial in silico diversity from full-length sequence reconstruction cannot be excluded

## Evidence

- [readme] Supplementary data availability: [evidence span withheld — anchor status: external_doc]
- [methods] Database choice: [evidence span withheld — anchor status: not_found]
