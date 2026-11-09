---
name: longest-common-substring-computation
description: Use when choosing a read-recruitment overlap cutoff for metagenomic viral
  strain reconstruction and you need to show that cutoffs (e.g. 100-150 bp) separate
  within-species from cross-species sequence sharing.
when_to_use_negative:
- The viral genome of interest shares no conservation with any reference genome —
  the article notes a completely new virus sharing no conservation will not be detected
  by this reference-based approach
- The goal is read recruitment itself rather than cutoff justification — use the TAR-VIR
  overlap_extension module on FASTQ/SAM data instead of genome-vs-genome LCS analysis
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: TAR-VIR
  role: Pipeline whose overlap_extension read-recruitment stage applies the overlap
    cutoff that this LCS analysis empirically justifies
  repo: https://github.com/chjiao/TAR-VIR
- name: Python
  role: Implementation language for the LCS computation (generalized suffix array
    / dynamic programming) and ROC/AUC analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:57d4c14eca3a2aff6017e5509cd49a371cb0d219cbe15fb16094353e9b0d873f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chjiao/TAR-VIR
  tools:
  - Python
  - TAR-VIR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# longest-common-substring-computation

## Summary

Computes the longest common substring (LCS) size for each pair of viral (and virus-vs-other-microbe) genomes to empirically justify the overlap cutoff used in TAR-VIR's read recruitment, via an ROC curve whose AUC should be close to 1.

## When to use

Use when choosing a read-recruitment overlap cutoff for metagenomic viral strain reconstruction and you need to show that cutoffs (e.g. 100-150 bp) separate within-species from cross-species sequence sharing.

## When NOT to use

- The viral genome of interest shares no conservation with any reference genome — the article notes a completely new virus sharing no conservation will not be detected by this reference-based approach
- The goal is read recruitment itself rather than cutoff justification — use the TAR-VIR overlap_extension module on FASTQ/SAM data instead of genome-vs-genome LCS analysis

## Inputs

- Viral RefSeq genome records (e.g. HBV NC_003977.2, HCV genotypes NC_009827.1, NC_009823.1, NC_009825.1, NC_030791.1, NC_004102.1, NC_009826.1, NC_009824.1, HPgV NC_001710.1) from the NCBI viral genome database
- Microbial RefSeq genomes for virus-vs-other-microbe pairwise comparisons (~142 million pairs)

## Outputs

- LCS size distributions for within-quasispecies and cross-species genome pairs
- ROC curve and AUC quantifying separation of within-species vs cross-species LCS sizes
- lcs_roc_analysis.csv containing LCS size distributions, key pair values (Vaccinia-Variola 469 bp, HCV g7-g5 154 bp), and ROC/AUC results

## How to apply

Download the reference genomes named in the article (e.g. HBV NC_003977.2, HCV genotypes NC_009827.1/NC_009823.1/NC_009825.1/NC_030791.1/NC_004102.1/NC_009826.1/NC_009824.1, HPgV NC_001710.1) from NCBI. Compute the LCS size for every pair of viral genomes and every virus-vs-other-microbe pair using an efficient algorithm such as a generalized suffix array or dynamic programming, as described in Additional file 1: Section 1. Build a ROC curve where, for each candidate LCS size l, the true positive rate is the probability that two strains within a quasispecies share an LCS of at least l and the false positive rate is the probability that two different species share an LCS of at least l; report the AUC (expected close to 1). Verify the extreme cross-species values: the largest between-virus LCS of 469 bp (Vaccinia vs Variola) and 154 bp (HCV genotype 7 vs genotype 5), and that only a small number of the ~142 million virus-vs-other pairs (e.g. 86 pairs) exceed 100 bp. The resulting distributions justify setting TAR-VIR's overlap threshold longer than half the read size (with a 150 bp cutoff used for 250 bp MiSeq reads) so chimeric or cross-species reads are not recruited.

## Related tools

- **TAR-VIR** (Pipeline whose overlap_extension read-recruitment stage applies the overlap cutoff that this LCS analysis empirically justifies) — https://github.com/chjiao/TAR-VIR
- **Python** (Implementation language for the LCS computation (generalized suffix array / dynamic programming) and ROC/AUC analysis)

## Evaluation signals

- Recomputed Vaccinia-Variola LCS equals the reported 469 bp
- Recomputed HCV genotype 7 vs genotype 5 LCS equals the reported 154 bp
- Only a small number of the ~142 million virus-vs-other pairs (e.g. 86 pairs) exceed a 100 bp LCS
- The ROC curve achieves an AUC close to 1 with near-zero false positive rate at overlap cutoffs above 100 bp
- With the resulting 150 bp overlap cutoff, all recruited reads in the simulated SARS-CoV/Influenza/HBV dataset are from SARS-CoV with no contamination

## Limitations

- A completely new virus that does not share any conservation with any reference genome will not be detected by methods relying on this reference-based overlap analysis
- No single recommended overlap threshold value is stated in the paper's discussion; only guidance derived from common-string-size analysis between species is provided

## Evidence

- [methods] Overlap cutoff rationale: "we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads"
- [results] Reference genome inputs: "We downloaded the reference genomes of HBV (NC_003977.2), multiple genotypes of HCV (NC_009827.1, NC_009823.1, NC_009825.1, NC_030791.1, NC_004102.1, NC_009826.1, NC_009824.1), and human pegivirus"
- [results] Specificity validation at the chosen cutoff: "Worth noting is all the recruited reads are from SARS-CoV (no contamination from Influenza and HBV)."
- [discussion] Cutoff guidance from common-string analysis: "We have provided guidance on choosing appropriate overlap cutoffs based on the analysis of the common string sizes between different species."
