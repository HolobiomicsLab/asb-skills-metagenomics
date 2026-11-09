---
name: genomic-region-sequence-handling
description: Use when you need per-region pairwise comparisons of homologous sequences
  (e.g., BLAST-derived region bins from metagenomic assemblies) quantified as synteny
  scores rather than SNP distances.
when_to_use_negative:
- Comparing sequences with extensive divergence below the BLAST detection thresholds
  (identity <97% or query coverage <70%), since no homologous region bins will exist
  to align
- Quantifying SNP-based strain differences — SynTracker is deliberately insensitive
  to SNPs and designed to detect structural changes; use an SNP-based tool such as
  inStrain or MIDAS instead
- Pairs of sequences whose cumulative aligned overlap is below 4,800 bp, which are
  excluded from downstream analysis
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: SynTracker
  role: Pipeline that fragments reference genomes, bins BLAST-hit regions, and orchestrates
    per-region pairwise synteny score computation
  repo: github.com/leylabmpi/SynTracker
- name: DECIPHER R package
  role: FindSynteny function performs the all-versus-all pairwise alignment within
    each region bin to identify syntenic blocks (maxGap and maxSep set to 15)
- name: BLAST
  role: Identifies homologous regions in target genomes/assemblies, whose hits with
    2-kbp flanks become the region bins
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:fb5e101628c4816ece7ba3d3d7ded59ac964254eaf9f365676161c9eba4fc75d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - DECIPHER R package
  - SynTracker
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# genomic-region-sequence-handling

## Summary

Handles homologous genomic region bins from SynTracker: within each bin (1-kbp central regions with 2-kbp flanks per BLAST hit), performs all-versus-all pairwise alignment with DECIPHER's FindSynteny to compute region-specific pairwise synteny scores used downstream for APSS calculation.

## When to use

Use when you need per-region pairwise comparisons of homologous sequences (e.g., BLAST-derived region bins from metagenomic assemblies) quantified as synteny scores rather than SNP distances.

## When NOT to use

- Comparing sequences with extensive divergence below the BLAST detection thresholds (identity <97% or query coverage <70%), since no homologous region bins will exist to align
- Quantifying SNP-based strain differences — SynTracker is deliberately insensitive to SNPs and designed to detect structural changes; use an SNP-based tool such as inStrain or MIDAS instead
- Pairs of sequences whose cumulative aligned overlap is below 4,800 bp, which are excluded from downstream analysis

## Inputs

- Region-specific bins: FASTA sequences per BLAST hit (central 1-kbp region with 2-kbp flanks) grouped by homologous region per reference genome
- BLAST hit coordinates from the SynTracker search (97% minimal identity, 70% minimal query coverage by default)
- Reference genome FASTA files fragmented into central regions

## Outputs

- Per-bin, per-pair TSV table with synteny score, syntenic block count, and cumulative aligned overlap length
- [genome name]_synteny_scores_per_region.csv raw per-region synteny score table (SynTracker output format)

## How to apply

Load the region-specific bins (central 1-kbp regions plus 2-kbp flanking sequences per BLAST hit, retrieved with blastcmddb) for each reference genome. Within each bin, run an all-versus-all pairwise alignment with the FindSynteny function in the DECIPHER R package, setting maxGap and maxSep both to 15, to identify syntenic blocks. Retain only pairwise comparisons whose cumulative aligned overlap reaches the minimal overlap threshold of 4,800 bp. For each retained pair compute synScore = 1 + log10((Ov/len)/B), where Ov is the cumulative aligned block length, len is the length of the shorter sequence, and B is the number of syntenic blocks; the score is maximal (1) for a single block with 100% overlap. Write a per-bin, per-pair TSV with score, block count, and overlap length, which feeds the region-level synteny_scores_per_region table used for APSS subsampling.

## Related tools

- **SynTracker** (Pipeline that fragments reference genomes, bins BLAST-hit regions, and orchestrates per-region pairwise synteny score computation) — github.com/leylabmpi/SynTracker
- **DECIPHER R package** (FindSynteny function performs the all-versus-all pairwise alignment within each region bin to identify syntenic blocks (maxGap and maxSep set to 15))
- **BLAST** (Identifies homologous regions in target genomes/assemblies, whose hits with 2-kbp flanks become the region bins)

## Examples

```
Rscript -e "library(DECIPHER); synteny <- FindSynteny(bin_seqs, maxGap=15, maxSep=15)"
```

## Evaluation signals

- Output TSV exists and contains score, block-count, and overlap columns for every bin and every sequence pair
- All computed synteny scores are <= 1, with 1 expected only for a single syntenic block and 100% overlap
- Every retained pairwise comparison has cumulative aligned overlap >= 4,800 bp
- Per-genome raw table synteny_scores_per_region.csv matches the expected SynTracker output schema for downstream APSS subsampling

## Limitations

- Pairwise comparisons with minimal overlap below 4,800 bp are discarded, limiting coverage for divergent or poorly assembled targets
- Hits within <2 kbp both upstream and downstream are excluded, so regions near contig ends are not analyzed
- Synteny scores are sensitive to block count and overlap but not to point mutations; strains differing mostly by SNPs will score similarly

## Evidence

- [discussion] using the ‘FindSynteny’ function in the DECIPHER R package21, with parameters ‘maxGap’ and ‘maxSep’ both set to 15: "using the ‘FindSynteny’ function in the DECIPHER R package21, with parameters ‘maxGap’ and ‘maxSep’ both set to 15"
- [discussion] minimal overlap threshold: "only pairwise comparisons with a minimal overlap of 4,800 bp are considered for downstream analysis"
- [discussion] region extraction with flanks: "a 2-kbp region on each side of the BLAST hit using the ‘blastcmddb’ command"
- [discussion] fragmentation into central regions: "Each per-species reference is fragmented into a collection of 1-kbp central regions, which are binned and stored together."
- [readme] per-region output table: "The table `[genome name]_synteny_scores_per_region.csv` contains the raw results obtained by the comparison of each two homologous genomic"
