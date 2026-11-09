---
name: classification-accuracy-metrics-evaluation
description: Use when you have per-read classification labels (C/U/A/H plus TaxID
  and maxSim) from LiME or a comparable classifier on simulated metagenomes with known
  ground-truth taxonomic provenance, and you need SEN/PREC/F1/specificity at a fixed
  taxonomic rank (species).
when_to_use_negative:
- Input is a real metagenome without ground-truth species labels (e.g. SRR1804065)
  — accuracy metrics like SEN/PREC against truth cannot be computed; use concordance/agreement
  measures (e.g. Jaccard-based rid and ras) instead
- The goal is to evaluate LiME's computational performance (time/memory) rather than
  classification accuracy — the article defers those results to supplementary Table
  S3
- Classification was run at a taxonomic rank other than species (TaxRank != 1) — the
  published Table S1 benchmark values correspond to species-level classification
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: LiME
  role: Source of the classification output (C/U/A/H,IdSeqRead,TaxID,maxSim) whose
    SEN/PREC/F1/specificity are computed; run in both EBWT=1 (LiMEeBWT) and EBWT=0
    (LiMEDA) variants
  repo: https://github.com/veronicaguerrini/LiME
- name: MagicBlast
  role: Benchmark classifier whose published accuracy values are compared against
    LiME's computed metrics
- name: CLARK-S
  role: Benchmark classifier whose published accuracy values are compared against
    LiME's computed metrics
- name: Centrifuge
  role: Benchmark classifier whose published accuracy values are compared against
    LiME's computed metrics
- name: Kraken 2
  role: Benchmark classifier whose published accuracy values are compared against
    LiME's computed metrics
- name: TaxMaps
  role: Benchmark classifier whose published accuracy values are compared against
    LiME's computed metrics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:b5669be6e6d1c7387b45bd15b4a4c5a5fe2ecc93ae3a9b92d9e341c26ed21917
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR
  - CLARK-S
  - Centrifuge
  - Kraken 2
  - LiME
  - MagicBlast
  - TaxMaps
  - eGAP
  - eGSA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# classification-accuracy-metrics-evaluation

## Summary

Computes species-level classification accuracy (sensitivity SEN, precision PREC, F1 score, specificity) from LiME's comma-separated classification output on the simulated paired-end metagenomes setA2 and setB2, and benchmarks it against Table S1 values for Magic-BLAST, CLARK-S, Centrifuge, Kraken 2 and TaxMaps.

## When to use

Use when you have per-read classification labels (C/U/A/H plus TaxID and maxSim) from LiME or a comparable classifier on simulated metagenomes with known ground-truth taxonomic provenance, and you need SEN/PREC/F1/specificity at a fixed taxonomic rank (species).

## When NOT to use

- Input is a real metagenome without ground-truth species labels (e.g. SRR1804065) — accuracy metrics like SEN/PREC against truth cannot be computed; use concordance/agreement measures (e.g. Jaccard-based rid and ras) instead
- The goal is to evaluate LiME's computational performance (time/memory) rather than classification accuracy — the article defers those results to supplementary Table S3
- Classification was run at a taxonomic rank other than species (TaxRank != 1) — the published Table S1 benchmark values correspond to species-level classification

## Inputs

- Comma-separated LiME OutputFile with entries 'C/U/A/H,IdSeqRead,TaxID,maxSim' per read
- Ground-truth species provenance per read from the curated Lindgreen et al. simulated datasets setA2 and setB2
- Reference_database.csv taxonomy file for the 930-genome reference database Refs.fasta
- Published benchmark values (Table S1) for Magic-BLAST, CLARK-S, Centrifuge, Kraken 2 and TaxMaps

## Outputs

- accuracy_table.csv with SEN, PREC, F1, specificity, and counts of ambiguous/unclassified reads per classifier per dataset, reproducing the Table S1 layout
- Per-read status labels (one of C/U/A/H) per read pair for both LiME variants on both datasets

## How to apply

Parse the comma-separated OutputFile with entries 'C/U/A/H,IdSeqRead,TaxID,maxSim' produced by Classify run with TaxRank=1 (species) and the LineageFile in the format Accession_number;Species_TaxID;Genus_TaxID;Family_TaxID;Order_TaxID;Class_TaxID;Phylum_TaxID. Compute sensitivity (SEN), precision (PREC), F1 score, specificity, and counts of ambiguous and unclassified reads by matching assigned TaxIDs to the ground-truth species of each read in the curated Lindgren et al. datasets (Eukaryote reads with unspecified species removed, and reads from 3 genomes with expiring NCBI entries excluded). Do this separately for both LiME variants — LiMEeBWT (EBWT=1, reads da and ebwt) and LiMEDA (EBWT=0, reads only da) — and for both setA2 and setB2. Assemble the metrics into an accuracy_table.csv reproducing the layout of supplementary Table S1 alongside the published values for Magic-BLAST, CLARK-S, Centrifuge, Kraken 2 and TaxMaps. Validation relies on consistency with Table S1: LiME should show very high precision and specificity, with 99.9% of positive control reads correctly assigned and less than 0.01% of negative control reads classified.

## Related tools

- **LiME** (Source of the classification output (C/U/A/H,IdSeqRead,TaxID,maxSim) whose SEN/PREC/F1/specificity are computed; run in both EBWT=1 (LiMEeBWT) and EBWT=0 (LiMEDA) variants) — https://github.com/veronicaguerrini/LiME
- **MagicBlast** (Benchmark classifier whose published accuracy values are compared against LiME's computed metrics)
- **CLARK-S** (Benchmark classifier whose published accuracy values are compared against LiME's computed metrics)
- **Centrifuge** (Benchmark classifier whose published accuracy values are compared against LiME's computed metrics)
- **Kraken 2** (Benchmark classifier whose published accuracy values are compared against LiME's computed metrics)
- **TaxMaps** (Benchmark classifier whose published accuracy values are compared against LiME's computed metrics)

## Examples

```
Classify 4 Reads1F+Refs_clusterOut1 ... Reads2RC+Refs_clusterOut4 20249373 930 OutputFile.csv Reference_database_lineage.csv 1 threads
```

## Evaluation signals

- Reproduced SEN/PREC/F1/specificity for LiME are consistent with Table S1 of the paper (very high precision and specificity)
- 99.9% of positive control reads are correctly assigned and less than 0.01% of negative control reads are classified
- Every read pair yields exactly one status label (C/U/A/H) in the parsed OutputFile
- LiMEeBWT and LiMEDA both show higher precision than TaxMaps, CLARK-S and Centrifuge on both setA2 and setB2
- accuracy_table.csv layout matches supplementary Table S1, including benchmark columns for Magic-BLAST, CLARK-S, Centrifuge, Kraken 2 and TaxMaps

## Limitations

- Version numbers of the compared tools (Magic-BLAST, CLARK-S, Centrifuge, Kraken 2 and TaxMaps) are not stated in the article
- The comparison uses curated simulated datasets: Eukaryote reads with unspecified species provenance and reads from 3 genomes with expiring NCBI entries were removed, so metrics do not reflect the original raw datasets
- Parameter sweep details for the evaluation are deferred to supplementary Table S2 and full computational performance results for setA2 to Table S3, not included in the main section

## Evidence

- [results] On both simulated datasets (setA2 and setB2), LiMEeBWT and LiMEDA show higher precision than TaxMaps, CLARK-S and Centrifuge: [evidence span withheld — anchor status: not_found]
- [abstract] Positive/negative control accuracy figures: "99.9% of the positive control reads are correctly assigned and the percentage of classified reads of the negative control is less than 0.01%"
- [results] Metric computation from LiME output: "from which we can calculate the quality metrics: sensitivity SEN, precision PREC, and the F1 score"
- [results] Species-level rank setting: "For the experiments reported in Figs. 2 and 3, we set the taxonomic rank of classification to species"
- [readme] Output file format of Classify: "OutputFile is the comma-separated file where the classification results will be stored according to the format: *C/U/A/H,IdSeqRead,TaxID,maxSim*"
- [results] Dataset curation of simulated sets: "we first removed a group of reads associated with the phylum of Eukaryotes whose species provenance was not specified"
