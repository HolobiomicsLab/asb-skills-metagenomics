---
name: synteny-block-alignment-with-decipher
description: Use when you already have region-specific bins of homologous sequences
  (e.g., central 1-kbp regions plus 2-kbp flanks per BLAST hit) and need per-pair
  synteny scores based on synteny-block structure rather than SNPs.
when_to_use_negative:
- You need SNP-based strain comparison; this skill is insensitive to point mutations
  and instead captures structural differences via synteny
- Input is raw reads or unassembled metagenomes — SynTracker operates on metagenomic
  assemblies or genomes, not reads
- Pairwise sequence pairs fall below the 4,800 bp minimal overlap and would be excluded
  from downstream APSS averaging
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: SynTracker
  role: Pipeline that bins homologous regions from BLAST hits and invokes DECIPHER-based
    synteny scoring within each bin
  repo: https://github.com/leylabmpi/SynTracker
- name: DECIPHER R package
  role: Provides the FindSynteny function used for all-versus-all pairwise alignment
    to identify syntenic blocks (maxGap and maxSep both set to 15)
- name: BLAST
  role: Upstream BLAST search (minimal identity 97%, minimal query coverage 70%) that
    defines the homologous-region bins scored by this skill
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:d2f6c506f83d4c79af069cfd2ef736153099457eb32eb77dc2876a31b4a38f13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - DECIPHER R package
  - SynTracker
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# synteny-block-alignment-with-decipher

## Summary

Computes region-specific pairwise synteny scores within each bin of homologous regions (derived from the same BLAST search) by running all-versus-all alignment with the DECIPHER R package's FindSynteny and scoring each pair from its synteny-block count and aligned overlap. This step supplies the raw per-region scores that SynTracker later averages into APSS values.

## When to use

Use when you already have region-specific bins of homologous sequences (e.g., central 1-kbp regions plus 2-kbp flanks per BLAST hit) and need per-pair synteny scores based on synteny-block structure rather than SNPs.

## When NOT to use

- You need SNP-based strain comparison; this skill is insensitive to point mutations and instead captures structural differences via synteny
- Input is raw reads or unassembled metagenomes — SynTracker operates on metagenomic assemblies or genomes, not reads
- Pairwise sequence pairs fall below the 4,800 bp minimal overlap and would be excluded from downstream APSS averaging

## Inputs

- Region-specific bins of homologous sequences (central 1-kbp regions with 2-kbp flanks) per reference genome, derived from BLAST hits (>=97% identity, >=70% query coverage)
- Reference genome FASTA files (complete genomes or single-file collections of contigs)

## Outputs

- Per-bin, per-pair TSV table with synteny score, syntenic-block count, and cumulative aligned overlap length for each sequence pair (raw synteny_scores_per_region.csv in SynTracker output)

## How to apply

Load each region-specific bin produced by the upstream BLAST step (central 1-kbp regions with 2-kbp flanking sequences per reference genome). Within each bin, run all-versus-all pairwise alignment with the FindSynteny function of the DECIPHER R package, setting maxGap and maxSep both to 15, to identify syntenic blocks between every pair of sequences. Retain only pairwise comparisons whose cumulative aligned overlap reaches the minimal overlap threshold of 4,800 bp. For each retained pair compute synScore = 1 + log10((Ov/len)/B), where Ov is the cumulative aligned block length, len is the length of the shorter sequence, and B is the number of syntenic blocks; the score is maximally 1 for a single block with 100% overlap. Write a per-bin, per-pair TSV recording the synteny score, block count, and overlap length for each sequence pair.

## Related tools

- **SynTracker** (Pipeline that bins homologous regions from BLAST hits and invokes DECIPHER-based synteny scoring within each bin) — https://github.com/leylabmpi/SynTracker
- **DECIPHER R package** (Provides the FindSynteny function used for all-versus-all pairwise alignment to identify syntenic blocks (maxGap and maxSep both set to 15))
- **BLAST** (Upstream BLAST search (minimal identity 97%, minimal query coverage 70%) that defines the homologous-region bins scored by this skill)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- Output TSV exists and contains score, block-count, and overlap columns for every bin and sequence pair
- All computed synteny scores are <= 1, since the maximal value is 1 for a single block with 100% overlap
- Every retained pairwise comparison has cumulative aligned overlap >= 4,800 bp
- FindSynteny was run with maxGap = 15 and maxSep = 15 within each region-specific bin
- Downstream APSS tables ([genome name]_avg_synteny_scores_[N]_regions.csv) can be reproduced by averaging these per-region scores over N subsampled regions

## Limitations

- SynTracker is designed to be sensitive to structural changes and has low sensitivity to SNPs, so this skill will not detect strains diverging mainly by point mutations
- Pairwise comparisons below the 4,800 bp minimal overlap threshold are discarded, reducing coverage for divergent or fragmented regions
- Strain delineation thresholds derived from these scores (e.g., APSS 0.95) are subjective and species-dependent; the authors recommend avoiding binary same/different strain calls
- With few regions per pairwise comparison, accuracy mostly decreases; a trade-off exists between accuracy and sample size

## Evidence

- [discussion] using the ‘FindSynteny’ function in the DECIPHER R package21, with parameters ‘maxGap’ and ‘maxSep’ both set to 15: "using the ‘FindSynteny’ function in the DECIPHER R package21, with parameters ‘maxGap’ and ‘maxSep’ both set to 15"
- [discussion] only pairwise comparisons with a minimal overlap of 4,800 bp are considered for downstream analysis: "only pairwise comparisons with a minimal overlap of 4,800 bp are considered for downstream analysis"
- [discussion] per each pairwise alignment, a synteny score is calculated: "per each pairwise alignment, a synteny score is calculated"
- [readme] The table [genome name]_synteny_scores_per_region.csv contains the raw results obtained by the comparison of each two homologous genomic regions: "The table `[genome name]_synteny_scores_per_region.csv` contains the raw results obtained by the comparison of each two homologous genomic "
