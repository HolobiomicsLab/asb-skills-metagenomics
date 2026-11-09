---
name: 16s-rrna-insertion-boundary-detection
description: Use when you have curated full-length 16S rRNA gene sequences (e.g. ≥800
  bp aligned bases) and need to flag which carry ≥10 bp insertions and report max/mean/sd
  insertion lengths, or when replicating the Brown et al. 2015 (10.1038/nature14486)
  CPR insertion statistics.
when_to_use_negative:
- Sequences are shorter than the ≥800 bp aligned-sequence threshold the paper used
  for phylogenetic/insertion analysis — boundaries will be unreliable
- The goal is insertion detection in transcribed rRNAs — the paper's metatranscriptomic
  mapping showed insertions are not retained in transcribed RNAs and are probably
  rapidly degraded
- You need insertion detection in eukaryotic or archaeal rRNAs, or in non-CPR bacterial
  sets, where the paper found such insertions are rare (SILVA v115 analysis)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Rfam (searched with cmscan)
  role: Covariance-model alignment of full 16S rRNA sequences used to locate large
    alignment gaps defining insertion boundaries
- name: Comparative RNA Web seed alignment
  role: Source of a manually curated seed alignment from which a sequence-only rRNA
    model was built for the insertion search
- name: USEARCH (cluster_smallmem)
  role: Clustering assembled 16S rRNA gene sequences at 97% sequence identity to remove
    redundancy before insertion statistics are computed
- name: fix_assembly_errors (re_assemble_errors.py / ra2.py)
  role: Curation of assembled scaffolds to eliminate assembly errors so that apparent
    alignment gaps reflect true insertions, not misassemblies
  repo: https://github.com/christophertbrown/fix_assembly_errors
- name: pandas/numpy
  role: Lightweight Python computation of the fraction-inserted and max/mean/sd insertion-length
    statistics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:953113ea8062ba53cef8a1b5625053dda2c25184500fb3e4a40dc924229e09e7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/christophertbrown/fix_assembly_errors
  tools:
  - Comparative RNA Web seed alignment
  - Rfam (searched with cmscan)
  - USEARCH (cluster_smallmem)
  - fix_assembly_errors (re_assemble_errors.py / ra2.py)
  - pandas/numpy
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# 16s-rrna-insertion-boundary-detection

## Summary

Detects and characterizes large (≥10 bp) insertion sequences within bacterial 16S rRNA genes by identifying large gaps in HMM/covariance-model alignments (e.g. cmscan against Rfam or a sequence-only model built from the Comparative RNA Web seed alignment), then computes fraction-inserted and insertion-length statistics as a structured JSON record.

## When to use

Use when you have curated full-length 16S rRNA gene sequences (e.g. ≥800 bp aligned bases) and need to flag which carry ≥10 bp insertions and report max/mean/sd insertion lengths, or when replicating the Brown et al. 2015 (10.1038/nature14486) CPR insertion statistics.

## When NOT to use

- Sequences are shorter than the ≥800 bp aligned-sequence threshold the paper used for phylogenetic/insertion analysis — boundaries will be unreliable
- The goal is insertion detection in transcribed rRNAs — the paper's metatranscriptomic mapping showed insertions are not retained in transcribed RNAs and are probably rapidly degraded
- You need insertion detection in eukaryotic or archaeal rRNAs, or in non-CPR bacterial sets, where the paper found such insertions are rare (SILVA v115 analysis)

## Inputs

- Curated 16S rRNA gene sequence set (bacterial genes ≥800 bp of aligned sequence, e.g. Supplementary Data 2 with 1,543 genes)
- rRNA insertion catalog from HMM/covariance-model alignment gap analysis (e.g. Supplementary Table 5)
- rRNA covariance model / reference database (Rfam, or a sequence-only model from the Comparative RNA Web seed alignment)

## Outputs

- Per-gene flags indicating presence of a ≥10 bp insertion (number of flagged genes / total 1,543)
- Insertion length statistics: fraction inserted, maximum (bp), mean (bp), sample standard deviation (bp)
- Single structured JSON record containing the four metrics

## How to apply

First assemble and curate 16S rRNA genes to eliminate assembly errors (e.g. with the fix_assembly_errors re_assemble_errors.py/ra2.py pipeline) and cluster at 97% sequence identity with USEARCH cluster_smallmem to remove redundancy, keeping only genes with ≥800 bp of aligned sequence. Align each full rRNA sequence against an rRNA covariance model — the paper searched full rRNA sequences against Rfam using cmscan, or built a sequence-only model from the manually curated Comparative RNA Web seed alignment — and define insertion boundaries as large gaps in the model alignment. Apply the ≥10 bp length cutoff, mirroring the paper's rule that all insertions ≥10 bp were removed before multiple sequence alignment and phylogenetic analysis. Compute the fraction of genes carrying a large insertion and the maximum, mean, and sample standard deviation of insertion lengths with a lightweight Python computation (pandas/numpy), then emit the four metrics as one JSON record.

## Related tools

- **Rfam (searched with cmscan)** (Covariance-model alignment of full 16S rRNA sequences used to locate large alignment gaps defining insertion boundaries)
- **Comparative RNA Web seed alignment** (Source of a manually curated seed alignment from which a sequence-only rRNA model was built for the insertion search)
- **USEARCH (cluster_smallmem)** (Clustering assembled 16S rRNA gene sequences at 97% sequence identity to remove redundancy before insertion statistics are computed)
- **fix_assembly_errors (re_assemble_errors.py / ra2.py)** (Curation of assembled scaffolds to eliminate assembly errors so that apparent alignment gaps reflect true insertions, not misassemblies) — https://github.com/christophertbrown/fix_assembly_errors
- **pandas/numpy** (Lightweight Python computation of the fraction-inserted and max/mean/sd insertion-length statistics)

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('insertions.tsv',sep='\t'); b=df[df.length>=10]; print(len(b)/1543, b.length.max(), b.length.mean(), b.length.std())"
```

## Evaluation signals

- Fraction of the 1,543 curated genes flagged as carrying a ≥10 bp insertion reproduces the reported 31% within rounding tolerance
- Maximum insertion length reproduces 2,004 bp; mean reproduces 519 bp; sample standard deviation reproduces 372 bp
- No gene with <800 bp of aligned sequence contributes to the statistics (mirrors the ≥800 bp inclusion rule)
- Every insertion ≥10 bp is flagged and removed before any downstream multiple sequence alignment / phylogeny is built
- Output is a single structured JSON record containing exactly the four metrics (fraction, max, mean, s.d.)

## Limitations

- Insertion boundaries are defined by large gaps in the HMM/covariance-model alignment, so model quality and the curated Comparative RNA Web seed alignment directly affect boundary accuracy
- Statistics are calibrated on a CPR-enriched, groundwater-derived gene set; an extensive search of the Silva database suggested insertions are rare in bacteria outside the CPR, so 31% will not generalize
- Insertions are probably self-splicing introns — they are not retained in transcribed RNAs — so detected 'insertions' are genomic features, not transcript features

## Evidence

- [abstract] Surprisingly, 31% of 16S rRNA genes encoded a large (≥10 bp) insertion sequence (maximum 2,004 bp; mean 519 bp; standard deviation (s.d.) 372 bp: "Surprisingly, 31% of 16S rRNA genes encoded a large (≥10 bp) insertion sequence (maximum 2,004 bp; mean 519 bp; standard deviation (s.d.) 372 bp"
- [methods] all insertions ≥10 bp were removed before multiple sequence alignment: "all insertions ≥10 bp were removed before multiple sequence alignment and phylogenetic analysis of 16S rRNA gene sequences"
- [methods] searching full rRNA sequences against Rfam using cmscan: [evidence span withheld — anchor status: not_found]
- [methods] we built a sequenceonly model from the manually curated seed alignment maintained by the Comparative RNA Web: "we built a sequenceonly model from the manually curated seed alignment maintained by the Comparative RNA Web"
- [abstract] Mapping metatranscriptomic sequences to assembled 16S rRNA genes showed that insertions are not retained in transcribed RNAs and are probably rapidly degraded: "Mapping metatranscriptomic sequences to assembled 16S rRNA genes showed that insertions are not retained in transcribed RNAs and are probably rapidly degraded"
