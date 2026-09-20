---
name: classification-performance-metrics-computation
description: Use when evaluating a taxonomic classifier against ground-truth community
  composition (e.g. a mock community or simulated human gut dataset) and you need
  genus/species-level precision, recall, F-measure and runtime comparisons.
when_to_use_negative:
- Input is real environmental metagenomic data without ground-truth composition —
  precision/recall against true taxa cannot be computed; novelty discovery is the
  appropriate skill instead
- You only need to run and profile a single classifier rather than compare multiple
  tools on identical inputs
- Fast read-level classification throughput is the goal — RiboTaxa benchmarking targets
  accuracy, and RiboTaxa itself is slower than short-read classifiers
edam_operation: http://edamontology.org/operation_0372
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: RiboTaxa
  role: 'Pipeline under evaluation: trims reads (BBTools), filters 16S/18S with SortMeRNA,
    reconstructs full-length SSU rRNA with EMIRGE+MetaRib, classifies with QIIME2
    sklearn at confidence 0.7'
  repo: https://github.com/oschakoory/RiboTaxa
- name: Kraken2
  role: Competing short-read classifier benchmarked on the same mock and synthetic
    gut datasets
- name: Centrifuge
  role: Competing short-read classifier benchmarked on the same datasets
- name: SPINGO
  role: Competing species-level amplicon classifier included in the benchmark
  repo: https://github.com/GuyAllard/SPINGO
- name: QIIME 2 Sklearn classifier
  role: Classifies full-length reconstructed SSU sequences within RiboTaxa (SILVA
    138 trained classifier, confidence 0.7)
  repo: https://github.com/oschakoory/RiboTaxa
- name: SortMeRNA
  role: Filters 16S/18S reads from metagenomic inputs prior to reconstruction and
    classification
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:caf36c48bc529211dc617db827ab8db078e4827252ab7732abec64d5e5ec66f1
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
  edam_operation: http://edamontology.org/operation_0372
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# classification-performance-metrics-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks taxonomic classifiers by comparing their genus- and species-level calls on mock or synthetic communities of known composition, computing precision, recall and F-measure per tool plus wall-clock runtime, as done for RiboTaxa versus Kraken2, Centrifuge, METAXA2, phyloFlash, SPINGO, BLCA and MEGAN6.

## When to use

Use when evaluating a taxonomic classifier against ground-truth community composition (e.g. a mock community or simulated human gut dataset) and you need genus/species-level precision, recall, F-measure and runtime comparisons.

## When NOT to use

- Input is real environmental metagenomic data without ground-truth composition — precision/recall against true taxa cannot be computed; novelty discovery is the appropriate skill instead
- You only need to run and profile a single classifier rather than compare multiple tools on identical inputs
- Fast read-level classification throughput is the goal — RiboTaxa benchmarking targets accuracy, and RiboTaxa itself is slower than short-read classifiers

## Inputs

- Metagenomic Illumina FASTQ reads from a mock community and a synthetic human gut community (e.g. HG_R1/HG_R2)
- Known (ground-truth) community composition at genus and species level
- Taxonomic profile outputs from each classifier (RiboTaxa, Kraken2, Centrifuge, METAXA2, phyloFlash BS/BE/SE, SPINGO, BLCA, MEGAN6)
- Per-tool wall-clock runtime records

## Outputs

- benchmark_performance_table.csv with precision, recall and F-measure per tool at genus and species level per dataset
- benchmark_runtime_table.csv with wall-clock runtime per tool per dataset
- Comparison figures equivalent to article Figure 8

## How to apply

Run each classifier (RiboTaxa pipeline: BBTools trimming, SortMeRNA 16S/18S filtering, EMIRGE with JOIN_THRESHOLD identity=1 and MIN_COV=0.3 combined with MetaRib, and QIIME2 sklearn classification at confidence 0.7, plus each competitor) on the same mock and synthetic human gut FASTQ inputs using an indexed SILVA SSU 138.1 database. Parse each tool's genus- and species-level taxonomic calls and compare them to the known community composition, counting true positives, false positives and false negatives per rank. Compute precision, recall and F-measure per tool per dataset and record wall-clock runtime for each tool-dataset pair. Assemble metrics into performance and runtime tables mirroring article Tables 2-3 and Figure 8. Validate by checking that the method of interest reproduces the reported F-measure (100% genus, 83.7% species for RiboTaxa on the synthetic gut community) with no false positives, and that runtimes fall in the reported minute-scale range (~18-26 min for RiboTaxa; short-read classifiers faster).

## Related tools

- **RiboTaxa** (Pipeline under evaluation: trims reads (BBTools), filters 16S/18S with SortMeRNA, reconstructs full-length SSU rRNA with EMIRGE+MetaRib, classifies with QIIME2 sklearn at confidence 0.7) — https://github.com/oschakoory/RiboTaxa
- **Kraken2** (Competing short-read classifier benchmarked on the same mock and synthetic gut datasets)
- **Centrifuge** (Competing short-read classifier benchmarked on the same datasets)
- **SPINGO** (Competing species-level amplicon classifier included in the benchmark) — https://github.com/GuyAllard/SPINGO
- **QIIME 2 Sklearn classifier** (Classifies full-length reconstructed SSU sequences within RiboTaxa (SILVA 138 trained classifier, confidence 0.7)) — https://github.com/oschakoory/RiboTaxa
- **SortMeRNA** (Filters 16S/18S reads from metagenomic inputs prior to reconstruction and classification) — https://github.com/oschakoory/RiboTaxa

## Examples

```
git clone https://github.com/oschakoory/RiboTaxa.git && cd RiboTaxa && bash conda_virt_env.sh
```

## Evaluation signals

- RiboTaxa reproduces F-measure of 100% at genus level and 83.7% at species level on the synthetic human gut community with zero false positives
- Per-tool runtimes fall in the reported minute-scale range (RiboTaxa ~18-26 min; Kraken2/Centrifuge faster)
- Metrics tables match the layout of article Tables 2-3 and Figure 8 for every tool-dataset pair
- Precision/recall/F-measure are computed against the exact known genus- and species-level composition of the mock and synthetic communities
- Classifications of reconstructed sequences show few false positives, whereas direct short-read classification inflates false positives

## Limitations

- Direct classification of short-length Illumina reads yields a high rate of false positives, so per-read classifiers may look worse for reasons of sequence length rather than database quality
- RiboTaxa is not as fast as short-read classifiers (i.e. Kraken2, Centrifuge), so runtime comparisons penalize reconstruction-based pipelines
- High conservation of 16S rRNA sequence cannot discriminate all species (e.g. certain Enterobacteriaceae, Lachnospiraceae), capping species-level recall for any classifier
- No 16S rRNA gene copy number normalization is applied, and artificial in silico diversity from full-length reconstruction cannot be excluded

## Evidence

- [supplementary] Synthetic gut benchmark data availability: [evidence span withheld — anchor status: external_doc]
