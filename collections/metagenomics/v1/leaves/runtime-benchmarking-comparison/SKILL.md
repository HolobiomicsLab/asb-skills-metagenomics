---
name: runtime-benchmarking-comparison
description: Use when a taxonomic classifier must be quantitatively validated against
  existing tools on mock or synthetic communities with known composition, requiring
  precision/recall/F-measure and runtime comparisons.
when_to_use_negative:
- The input is a real environmental or host-associated dataset with no known ground-truth
  composition — precision/recall and F-measure cannot be computed
- The goal is taxonomic profiling of a new sample rather than evaluating or comparing
  classifier performance
- Only short-read classifier speed matters and reconstruction-based classification
  accuracy is not of interest (RiboTaxa is not as fast as short-read classifiers)
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: RiboTaxa
  role: 'Pipeline under evaluation: BBTools trimming, SortMeRNA 16S/18S filtering,
    EMIRGE+MetaRib SSU reconstruction, QIIME2 sklearn classification'
  repo: https://github.com/oschakoory/RiboTaxa
- name: Kraken2
  role: Competing short-read taxonomic classifier benchmarked on the same datasets
- name: Centrifuge
  role: Competing short-read taxonomic classifier benchmarked on the same datasets
- name: SortMeRNA
  role: rRNA-specialized filter whose performance is compared against BBMap for extracting
    assemblable rRNA reads
- name: QIIME 2 Sklearn classifier
  role: Classifier used within RiboTaxa to classify reconstructed full-length SSU
    sequences at confidence 0.7
- name: SPINGO
  role: Competing amplicon species-level classifier included in the benchmark
  repo: https://github.com/GuyAllard/SPINGO
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:c1f71caaab450c2b09ab19c81abe8653a2b02006b1fff9d661e0fef5585ea6a0
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
  - Kraken2
  - QIIME 2 Sklearn classifier
  - RiboTaxa
  - SPINGO
  - SortMeRNA
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# runtime-benchmarking-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a new metagenomic taxonomic classifier (RiboTaxa) head-to-head against competing classifiers (Kraken2, Centrifuge, METAXA2, phyloFlash, SPINGO, BLCA, MEGAN6) on communities of known composition, recording genus/species-level F-measures and wall-clock runtimes into comparison tables.

## When to use

Use when a taxonomic classifier must be quantitatively validated against existing tools on mock or synthetic communities with known composition, requiring precision/recall/F-measure and runtime comparisons.

## When NOT to use

- The input is a real environmental or host-associated dataset with no known ground-truth composition — precision/recall and F-measure cannot be computed
- The goal is taxonomic profiling of a new sample rather than evaluating or comparing classifier performance
- Only short-read classifier speed matters and reconstruction-based classification accuracy is not of interest (RiboTaxa is not as fast as short-read classifiers)

## Inputs

- Illumina metagenomic FASTQ reads (single-end or paired-end, fastq.gz) from the MOCK community and simulated human gut community (HG_R1/HG_R2)
- Indexed SILVA SSU 138.1 reference database (SortMeRNA and Bowtie indices)
- QIIME2-trained SILVA 138 sklearn classifier
- Known (ground-truth) taxonomic composition of the benchmark communities

## Outputs

- benchmark_performance_table.csv with per-tool precision, recall and F-measure at genus and species level
- benchmark_runtime_table.csv with wall-clock runtime per tool per dataset
- Comparison plots reproducing article Tables 2-3 and Figure 8

## How to apply

Obtain the mock community and simulated human gut community (HG_R1/HG_R2) FASTQ reads plus the indexed SILVA SSU 138.1 database (SortMeRNA and Bowtie indices) and the QIIME2-trained SILVA 138 classifier. Run the RiboTaxa pipeline per dataset: BBTools adapter/quality trimming, SortMeRNA 16S/18S filtering, EMIRGE (JOIN_THRESHOLD identity = 1, 40 iterations, MIN_COV = 0.3) combined with MetaRib full-length SSU reconstruction, and QIIME2 sklearn classification at confidence 0.7. Run each competitor on the same inputs, then compare genus- and species-level calls to the known composition, computing precision, recall and F-measure per tool per dataset. Record wall-clock runtime for every tool–dataset pair and assemble benchmark_performance_table.csv and benchmark_runtime_table.csv matching article Tables 2-3 and Figure 8. The rationale for including reconstruction-based tools is that inferring taxa directly from short Illumina reads produces many false positives, whereas reconstructed full-length SSU sequences are correctly assigned.

## Related tools

- **RiboTaxa** (Pipeline under evaluation: BBTools trimming, SortMeRNA 16S/18S filtering, EMIRGE+MetaRib SSU reconstruction, QIIME2 sklearn classification) — https://github.com/oschakoory/RiboTaxa
- **Kraken2** (Competing short-read taxonomic classifier benchmarked on the same datasets)
- **Centrifuge** (Competing short-read taxonomic classifier benchmarked on the same datasets)
- **SortMeRNA** (rRNA-specialized filter whose performance is compared against BBMap for extracting assemblable rRNA reads)
- **QIIME 2 Sklearn classifier** (Classifier used within RiboTaxa to classify reconstructed full-length SSU sequences at confidence 0.7)
- **SPINGO** (Competing amplicon species-level classifier included in the benchmark) — https://github.com/GuyAllard/SPINGO

## Examples

```
git clone https://github.com/oschakoory/RiboTaxa.git; cd RiboTaxa; bash conda_virt_env.sh  # then run RiboTaxa and each competitor on MOCK and HG_R1/HG_R2 reads, timing each tool
```

## Evaluation signals

- RiboTaxa must reproduce the reported F-measure of 100% at genus level and 83.7% at species level on the simulated human gut community with no false positives
- Per-tool runtimes should fall in the reported minute-scale range (RiboTaxa ~18-26 min; short-read classifiers like Kraken2 and Centrifuge faster)
- Performance tables must match the column layout of article Tables 2-3 (per tool per dataset, genus and species level)
- Each tool's calls must be compared against the identical input FASTQ files and known community composition so precision/recall are directly comparable
- RiboTaxa output should report relative abundance description alongside taxonomic identifications, unlike competitors

## Limitations

- RiboTaxa is not as fast as short-read classifiers such as Kraken2 and Centrifuge
- Direct classification of short-length Illumina reads yields high false-positive rates, so short-read classifier results understate the true community when compared against reconstructed-sequence assignments
- 16S rRNA sequence conservation prevents species-level discrimination for some taxa (e.g. Enterobacteriaceae, Lachnospiraceae); RiboTaxa may stop at genus level
- No 16S rRNA gene copy number normalization is applied and artificial in silico diversity from full-length reconstruction cannot be excluded

## Evidence

- [methods] Classifier database used for the benchmark: [evidence span withheld — anchor status: not_found]
- [supplementary] Source of simulated gut benchmark data: [evidence span withheld — anchor status: external_doc]
