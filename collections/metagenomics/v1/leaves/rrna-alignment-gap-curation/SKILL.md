---
name: rrna-alignment-gap-curation
description: Use when you have full-length bacterial 16S rRNA gene sequences (≥800
  bp aligned) aligned to an rRNA covariance model and need to flag intron-like insertion
  sequences and quantify their length statistics before phylogenetic analysis.
when_to_use_negative:
- Sequences are short amplicon reads rather than near-full-length (≥800 bp aligned)
  16S rRNA genes — gap-based insertion calls are unreliable below this length
- You need insertions in eukaryotic or archaeal rRNAs — the paper's analysis and statistics
  target bacterial 16S rRNA genes, where insertions are rare outside the CPR
- Your goal is primer-binding or amplicon-detection assessment (e.g. 515F/806R coverage)
  rather than insertion detection and length statistics
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Rfam + cmscan (Infernal)
  role: Covariance-model search of full rRNA sequences used to detect insertion regions
    via alignment gaps
- name: Comparative RNA Web seed alignment
  role: Source of the manually curated alignment from which a sequence-only rRNA model
    was built
- name: fix_assembly_errors (re_assemble_errors.py / ra2.py)
  role: Upstream curation of assembled 16S rRNA gene scaffolds to eliminate assembly
    errors before insertion calling
  repo: https://github.com/christophertbrown/fix_assembly_errors
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:7bfd081db511f11511ffc4602588416ed4587de7734eaba09304d791bff67fb9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/christophertbrown/fix_assembly_errors
  tools:
  - Comparative RNA Web seed alignment
  - Rfam + cmscan (Infernal)
  - fix_assembly_errors (re_assemble_errors.py / ra2.py)
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# rrna-alignment-gap-curation

## Summary

Detect large insertions in curated 16S rRNA gene sequences by identifying big gaps in HMM/covariance-model alignments (e.g. cmscan against Rfam or a Comparative RNA Web-derived model), then compute the fraction of genes carrying ≥10 bp insertions and the max/mean/s.d. of insertion lengths.

## When to use

Use when you have full-length bacterial 16S rRNA gene sequences (≥800 bp aligned) aligned to an rRNA covariance model and need to flag intron-like insertion sequences and quantify their length statistics before phylogenetic analysis.

## When NOT to use

- Sequences are short amplicon reads rather than near-full-length (≥800 bp aligned) 16S rRNA genes — gap-based insertion calls are unreliable below this length
- You need insertions in eukaryotic or archaeal rRNAs — the paper's analysis and statistics target bacterial 16S rRNA genes, where insertions are rare outside the CPR
- Your goal is primer-binding or amplicon-detection assessment (e.g. 515F/806R coverage) rather than insertion detection and length statistics

## Inputs

- Curated 16S rRNA gene FASTA sequences (e.g. 1,543 bacterial genes ≥800 bp, Supplementary Data 2)
- rRNA covariance model / Rfam model or sequence-only model built from the Comparative RNA Web seed alignment
- Insertion catalog with per-gene insertion boundaries and lengths (e.g. Supplementary Table 5)

## Outputs

- Per-gene insertion flags and insertion length list (insertions ≥10 bp)
- Summary statistics: fraction of genes with large insertions, maximum, mean, and standard deviation of insertion lengths (reference: 31%, 2,004 bp, 519 bp, 372 bp)
- Intron-stripped 16S rRNA sequences ready for structural alignment and phylogeny
- Structured JSON record of the four metrics

## How to apply

Start from a curated set of bacterial 16S rRNA genes (in the source paper, 1,543 genes ≥800 bp, curated with fix_assembly_errors/re_assemble_errors.py and clustered at 97% identity with USEARCH). Align each full-length rRNA sequence to an rRNA model — the paper searched full rRNA sequences against Rfam using cmscan (Infernal) and built a sequence-only model from the Comparative RNA Web seed alignment. Call insertion boundaries from large gaps in the alignment between each sequence and the model, then apply the ≥10 bp cutoff so that all insertions ≥10 bp are removed before multiple sequence alignment and phylogenetic analysis. Finally compute the fraction of genes with ≥10 bp insertions and the maximum, mean, and sample standard deviation of insertion lengths; the paper's reference values were 31%, 2,004 bp, 519 bp, and 372 bp. Optionally annotate insertion-encoded ORFs by requiring ≥90% ORF overlap with the insertion and Rfam positive hits with ≥25% alignment overlap with the insertion.

## Related tools

- **Rfam + cmscan (Infernal)** (Covariance-model search of full rRNA sequences used to detect insertion regions via alignment gaps)
- **Comparative RNA Web seed alignment** (Source of the manually curated alignment from which a sequence-only rRNA model was built)
- **fix_assembly_errors (re_assemble_errors.py / ra2.py)** (Upstream curation of assembled 16S rRNA gene scaffolds to eliminate assembly errors before insertion calling) — https://github.com/christophertbrown/fix_assembly_errors

## Examples

```
big = df[df.ins_len >= 10]; print(big.gene.nunique()/1543, big.ins_len.max(), big.ins_len.mean(), big.ins_len.std())
```

## Evaluation signals

- Recomputed fraction of genes with ≥10 bp insertions matches 31% within rounding tolerance
- Maximum insertion length reproduces 2,004 bp; mean ≈ 519 bp; sample standard deviation ≈ 372 bp
- Every flagged insertion is ≥10 bp, mirroring the rule that all insertions ≥10 bp were removed before alignment and phylogenetic analysis
- Input gene set equals the curated 1,543 bacterial 16S rRNA genes ≥800 bp (post 97% USEARCH clustering and assembly-error curation)
- Metatranscriptomic read mapping shows insertions are not retained in transcribed RNAs, consistent with self-splicing intron interpretation

## Limitations

- Insertions are largely a CPR phenomenon — a search of the Silva database found them rare in bacteria outside the CPR, so the 31% rate will not generalize to typical bacterial gene sets
- Insertions are probably rapidly degraded self-splicing introns and are not retained in transcribed RNAs, so gene-presence statistics do not reflect the mature ribosome
- Divergent CPR 16S rRNA genes mean 50–100% of organisms in some phyla evade detection in typical surveys; coverage assumptions from non-CPR references may bias the gene set

## Evidence

- [methods] Rfam/cmscan search of full rRNA sequences: [evidence span withheld — anchor status: not_found]
