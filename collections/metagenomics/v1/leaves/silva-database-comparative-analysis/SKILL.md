---
name: silva-database-comparative-analysis
description: Use when quantifying large (≥10 bp) insertions in bacterial 16S rRNA
  gene sequences — e.g. comparing a CPR/CRISPR-adjacent genome set against SILVA 115
  reference sequences — or when auditing primer-amplicon bias caused by rRNA introns.
when_to_use_negative:
- The input is already a multiple sequence alignment or phylogenetic tree — insertions
  must be removed from raw gene sequences before alignment, not repaired afterwards.
- The question is whether insertions are retained in transcripts — use metatranscriptomic
  read mapping to assembled 16S rRNA genes instead; the paper shows insertions are
  not retained in transcribed RNAs.
- The target sequences are archaeal or eukaryotic rRNAs — the 31%/2,004 bp/519 bp/372
  bp statistics were derived for bacterial 16S rRNA genes and SILVA bacterial comparisons.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: cmscan (Rfam)
  role: Searching full rRNA sequences against Rfam to identify and characterize insertion
    regions
- name: SILVA 115
  role: Nonredundant reference 16S rRNA database analysed with the same insertion-detection
    methods for the outside-CPR comparison
- name: USEARCH
  role: Clustering assembled 16S rRNA gene sequences at 97% sequence identity to remove
    redundancy before analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:2a5735f89c5156b74cee24544b483e9f5d82060eda6b161689041f90b806afb6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SILVA 115
  - USEARCH
  - cmscan (Rfam)
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# silva-database-comparative-analysis

## Summary

Reproduce and extend the paper's rRNA-gene insertion survey by applying the same insertion-detection pipeline (HMM/covariance-model alignment gap calling, ≥10 bp cutoff) to the curated set of 1,543 bacterial 16S rRNA genes ≥800 bp and to nonredundant SILVA 115, showing that large insertions are rare in bacteria outside the CPR.

## When to use

Use when quantifying large (≥10 bp) insertions in bacterial 16S rRNA gene sequences — e.g. comparing a CPR/CRISPR-adjacent genome set against SILVA 115 reference sequences — or when auditing primer-amplicon bias caused by rRNA introns.

## When NOT to use

- The input is already a multiple sequence alignment or phylogenetic tree — insertions must be removed from raw gene sequences before alignment, not repaired afterwards.
- The question is whether insertions are retained in transcripts — use metatranscriptomic read mapping to assembled 16S rRNA genes instead; the paper shows insertions are not retained in transcribed RNAs.
- The target sequences are archaeal or eukaryotic rRNAs — the 31%/2,004 bp/519 bp/372 bp statistics were derived for bacterial 16S rRNA genes and SILVA bacterial comparisons.

## Inputs

- Curated set of 1,543 bacterial 16S rRNA gene sequences ≥800 bp (Supplementary Data 2)
- rRNA insertion catalog (Supplementary Table 5) with insertion boundaries from HMM/covariance-model alignment gaps
- SILVA version 115 nonredundant 16S rRNA reference sequences
- Comparative RNA Web seed alignment for building the sequence-only rRNA model

## Outputs

- Fraction of 16S rRNA genes carrying ≥10 bp insertions (expected 31%)
- Insertion length statistics: maximum 2,004 bp, mean 519 bp, standard deviation 372 bp
- Rfam/cmscan annotation table of insertion-encoded ORFs (positive hits: ≥25% alignment overlap with insertion; ≥90% ORF overlap with insertion)
- Insertion-stripped 16S rRNA sequences ready for structural multiple sequence alignment and phylogenetic analysis

## How to apply

Load the curated 16S rRNA gene set (1,543 bacterial genes ≥800 bp from Supplementary Data 2) and the rRNA insertion catalog (Supplementary Table 5), where insertion boundaries were defined by large gaps in the HMM/covariance-model alignment of each sequence to the rRNA model built from the Comparative RNA Web seed alignment; full rRNA sequences were searched against Rfam using cmscan, with hits called positive only if ≥25% of the alignment overlapped an insertion. Apply the paper's ≥10 bp cutoff (the same rule under which 'all insertions ≥10 bp were removed before multiple sequence alignment and phylogenetic analysis'). Compute the fraction of genes with large insertions and the maximum, mean, and sample standard deviation of insertion lengths (expected: 31%, 2,004 bp, 519 bp, 372 bp) with a lightweight pandas/numpy script. Then apply the identical procedure to version 115 of nonredundant SILVA — the paper states SILVA 'was analysed using the same methods' — to establish that insertions are rare outside the CPR.

## Related tools

- **cmscan (Rfam)** (Searching full rRNA sequences against Rfam to identify and characterize insertion regions)
- **SILVA 115** (Nonredundant reference 16S rRNA database analysed with the same insertion-detection methods for the outside-CPR comparison)
- **USEARCH** (Clustering assembled 16S rRNA gene sequences at 97% sequence identity to remove redundancy before analysis)

## Examples

```
import pandas as pd; ins = pd.read_csv('Supplementary_Table_5.tsv', sep='\t'); big = ins[ins.length >= 10]; print(len(big)/1543, big.length.max(), big.length.mean(), big.length.std())
```

## Evaluation signals

- Recomputed fraction of genes with ≥10 bp insertions equals 31% within rounding tolerance.
- Recomputed insertion length statistics match the reference values: maximum 2,004 bp, mean 519 bp, standard deviation 372 bp.
- All insertions ≥10 bp are absent from the sequences fed into the downstream multiple sequence alignment/phylogeny.
- Rfam/cmscan positive hits satisfy the ≥25% alignment-overlap-with-insertion criterion, and insertion-encoded ORFs satisfy the ≥90% ORF-overlap criterion.
- The SILVA 115 analysis uses the same model/alignment procedure and confirms insertions are rare in bacteria outside the CPR.

## Limitations

- Insertions are probably self-splicing introns: metatranscriptomic mapping showed insertions are not retained in transcribed RNAs and are probably rapidly degraded, so they affect gene surveys but not ribosome function.
- Only insertions ≥10 bp were catalogued and removed; smaller indels are not captured by this cutoff.
- Because of divergent 16S rRNA gene sequences in CPR, 50–100% of organisms sampled from specific phyla would evade detection in typical cultivation-independent surveys, and primers 515F/806R would probably miss ~20% of bacteria in this study — statistics may be sensitive to the gene set assembled.

## Evidence

- [methods] version 115 of nonredundant SILVA was analysed using the same methods: [evidence span withheld — anchor status: not_found]
- [methods] searching full rRNA sequences against Rfam using cmscan: [evidence span withheld — anchor status: not_found]
