---
name: metagenomic-classifier-benchmarking
description: Use when evaluating how a taxonomic classifier behaves on reads from
  species missing from its database, i.e. when quantifying false positives caused
  by similarity to the closest available relative rather than true species presence.
when_to_use_negative:
- The true source species IS present in the classifiers' databases — this skill measures
  false positives on absent species, not precision/recall on known species (use the
  3,269 MAG benchmark instead).
- The goal is abundance-estimation accuracy rather than read-level species misclassification,
  which the paper benchmarks separately on mock communities.
- Genomes cannot be verified as absent from the databases (AQ and ANI to closest relatives
  unknown), so a fair absence benchmark cannot be constructed.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: DeepMicrobes
  role: Deep-learning (k-mer embedding + LSTM + attention) species classifier under
    evaluation; produces far fewer false positives on absent-species reads
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: Kraken
  role: k-mer/LCA-based baseline classifier run with its standard database
- name: Kraken 2
  role: k-mer/LCA-based baseline classifier run with its standard database
- name: CLARK
  role: k-mer/alignment-based baseline classifier run with its standard database
- name: CLARK-S
  role: sensitive k-mer/alignment-based baseline classifier run with its standard
    database
- name: DeepMicrobes-data
  role: Provides benchmark datasets including reads simulated from species absent
    from reference databases, command lines used to run all benchmarked tools, and
    R scripts to generate figures
  repo: https://github.com/MicrobeLab/DeepMicrobes-data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:9e760917fd05ecd3e302d41cb6fcffcda753230f6107a69bea492d25366f6c8b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/MicrobeLab/DeepMicrobes
  repo_url: https://github.com/MicrobeLab/DeepMicrobes
  tools:
  - AFS-MetaCache
  - CLARK
  - CLARK-S
  - Centrifuge
  - DeepMicrobes
  - DeepMicrobes-data
  - Kraken
  - Kraken 2
  - Kraken2
  - Kraken2+Bracken
  - LiME
  - MagicBlast
  - MetaCacheSpark
  - TaxMaps
  - eGSA
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# metagenomic-classifier-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark taxonomic classifiers by simulating reads from genomes absent from all reference databases and comparing false positives (misclassified reads and misidentified species) produced by DeepMicrobes versus Kraken, Kraken 2, CLARK and CLARK-S. This isolates each tool's propensity to hallucinate species assignments for truly novel DNA.

## When to use

Use when evaluating how a taxonomic classifier behaves on reads from species missing from its database, i.e. when quantifying false positives caused by similarity to the closest available relative rather than true species presence.

## When NOT to use

- The true source species IS present in the classifiers' databases — this skill measures false positives on absent species, not precision/recall on known species (use the 3,269 MAG benchmark instead).
- The goal is abundance-estimation accuracy rather than read-level species misclassification, which the paper benchmarks separately on mock communities.
- Genomes cannot be verified as absent from the databases (AQ and ANI to closest relatives unknown), so a fair absence benchmark cannot be constructed.

## Inputs

- 121 genome assemblies from PRJNA348753 whose species are absent from all reference databases (query aligned 10-60%, ANI < 95% to closest relative)
- Standard classifier databases for DeepMicrobes, Kraken, Kraken 2, CLARK and CLARK-S
- Short-read simulator (for 1x coverage per-genome read simulation)

## Outputs

- Per-tool counts of misclassified reads and misidentified species (e.g. figure4_misclassification_results.csv, cf. Table S6)
- Grouped comparison plot reproducing the Figure 4 layout

## How to apply

From a genome set such as PRJNA348753, select genomes whose closest database relatives fall below novelty cutoffs (alignment coverage AQ<60% and ANI<95%) so that no tool's database contains the true source species. Simulate short reads at 1x coverage per genome with a short-read simulator, then classify each read set with DeepMicrobes (species model) and with k-mer/alignment baselines Kraken, Kraken 2, CLARK and CLARK-S, each using its standard database. Tally per tool the number of misclassified reads and the number of misidentified species, tabulate across the five classifiers, and render a grouped comparison plot in the Figure 4 layout. The key design point is that the comparison must hold across the whole range of similarity of each query genome to its closest database relative, since k-mer/LCA and alignment tools assign reads to the nearest database entry even when the true species is absent.

## Related tools

- **DeepMicrobes** (Deep-learning (k-mer embedding + LSTM + attention) species classifier under evaluation; produces far fewer false positives on absent-species reads) — https://github.com/MicrobeLab/DeepMicrobes
- **Kraken** (k-mer/LCA-based baseline classifier run with its standard database)
- **Kraken 2** (k-mer/LCA-based baseline classifier run with its standard database)
- **CLARK** (k-mer/alignment-based baseline classifier run with its standard database)
- **CLARK-S** (sensitive k-mer/alignment-based baseline classifier run with its standard database)
- **DeepMicrobes-data** (Provides benchmark datasets including reads simulated from species absent from reference databases, command lines used to run all benchmarked tools, and R scripts to generate figures) — https://github.com/MicrobeLab/DeepMicrobes-data

## Examples

```
mashdiff.sh -i query_genomes/ -r refseq.msh -s refseq -p gut  # filter genomes (AQ<60%, ANI<95%), then simulate 1x reads and classify with DeepMicrobes/Kraken/Kraken 2/CLARK/CLARK-S per DeepMicrobes-data command lines
```

## Evaluation signals

- DeepMicrobes produces far fewer misclassified reads than Kraken, Kraken 2, CLARK and CLARK-S across all bins of similarity (10-60% query aligned, ANI<95%) to the closest database relative
- DeepMicrobes produces far fewer misidentified species than all four baselines, consistent with the published Figure 4 / Table S6 result
- All 121 query genomes verified absent from every tool's database via AQ<60% and ANI<95% novelty filtering before classification
- Simulated read volume matches design: 1x coverage per genome from a short-read simulator
- Output table contains one row per classifier per genome (or aggregate) with both misclassified-read and misidentified-species counts

## Limitations

- Results depend on the novelty filtering step (AQ<60%, ANI<95%); genomes whose closest relatives sit above these cutoffs do not constitute a true absent-species benchmark
- Adding new species to DeepMicrobes requires retraining the entire deep neural network, so this benchmark cannot be generalized to novel-species inclusion without retraining
- The k-mer length used (12-mers in pre-trained models) is optimized for thousands of genomes; other scenarios may require trying different k-mer lengths (6-12 bp)
- Retraining might be required to adapt to reads with strongly different lengths and error profiles than the simulated short reads

## Evidence

- [other] the species model of DeepMicrobes produced far fewer false positives (misclassified reads and misidentified species) than Kraken, Kraken 2, CLARK and CLARK-S, regardless of the degree of similarity to the closest genome in the databases: [evidence span withheld — anchor status: not_found]
- [supplementary] Number of reads sampled from each whole-genome-sequenced bacterial isolates: [evidence span withheld — anchor status: external_doc]
- [readme] Reads simulated from species absent from reference databases: [evidence span withheld — anchor status: external_doc]
- [discussion] the k-mer length we used in this study is optimized for typical data volume of thousands of genomes: [evidence span withheld — anchor status: not_found]
