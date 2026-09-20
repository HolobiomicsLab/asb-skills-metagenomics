---
name: capture-probe-set-statistics
description: Use when a HUBDesign-style probe design run has completed and you need
  to report compactness (probe count), percent of input nucleotides covered, capture
  depth, and efficiency as in Table 1 of the Cell Reports Methods paper.
when_to_use_negative:
- The probe set has not yet been through Filtering, so counts would include baits
  that capture host (human GRCh38) sequences and inflate coverage
- The design was produced by a different pipeline with different bait lengths (e.g.,
  non-75 bp probes), making depth and efficiency values incomparable to the HUBDesign
  Table 1 figures
- You need wet-lab capture performance (fold enrichment of spiked targets), which
  this purely in-silico statistic does not predict — the paper notes enrichment is
  target-sequence dependent
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: HUBDesign
  role: Produces the probe set (Clustering, Assignment, Identification, Filtering,
    Selection) whose final FASTA is counted and characterized by these statistics
  repo: https://github.com/zacherydickson/HUBDesign
- name: BLASTn
  role: Used during the Filtering phase against blacklist databases (e.g., human GRCh38)
    so that only host-safe probes enter the counted set
- name: PROKKA
  role: Suggested annotation tool generating the annotated genome inputs from which
    total input nucleotides are derived
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:8f7abab36720e31506ed6e84ad8e34c05d6d26327b51c857646354ef1680b12b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/zacherydickson/HUBDesign
  repo_url: https://github.com/zacherydickson/HUBDesign
  tools:
  - BLASTn
  - BioPerl
  - CLUSTAL OMEGA
  - HUBDesign
  - MAFFT
  - MUSCLE
  - PROKKA
  - Parallel::ForkManager
  - Perl
  - dustmasker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# capture-probe-set-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the size and efficiency of a hybridization-capture probe set produced by HUBDesign (e.g., the 26,870-probe sepsis set), computing target coverage, average capture depth, and probes-per-target efficiency for comparison with tiling strategies.

## When to use

Use when a HUBDesign-style probe design run has completed and you need to report compactness (probe count), percent of input nucleotides covered, capture depth, and efficiency as in Table 1 of the Cell Reports Methods paper.

## When NOT to use

- The probe set has not yet been through Filtering, so counts would include baits that capture host (human GRCh38) sequences and inflate coverage
- The design was produced by a different pipeline with different bait lengths (e.g., non-75 bp probes), making depth and efficiency values incomparable to the HUBDesign Table 1 figures
- You need wet-lab capture performance (fold enrichment of spiked targets), which this purely in-silico statistic does not predict — the paper notes enrichment is target-sequence dependent

## Inputs

- Final probe FASTA produced by HUBDesign Selection (TiledBaits.fna)
- Tab-delimited file describing the source and position of each probe sequence
- Input genome statistics (total nucleotides across the 1,926 annotated PATRIC bacterial genomes)
- Selected tiling density (5 bp) and probe-count cap used during Selection

## Outputs

- Probe count for the final set (e.g., 26,870 probes)
- Target coverage as percent of all input nucleotides (e.g., 2.09%)
- Average depth of coverage (e.g., 3.64x)
- Efficiency metric (e.g., 29.3) suitable as a Table 1 statistics row

## How to apply

After running the HUBDesign pipeline (Clustering at 95% penetrance, Assignment, Identification of up to 100 candidate 100 bp bait regions per pseudo-genome cut into 75 bp sub-probes, Filtering against a BLASTn blacklist such as GRCh38, and Selection at 5 bp tiling density with a probe-count cap), count the records in the final probe FASTA. Compute target coverage as the fraction of all input nucleotides spanned by probes, average depth as the mean number of probes tiling each covered base, and efficiency as targets captured per probe. For the 1,926-genome PATRIC sepsis dataset the published reference values are 26,870 probes, 2.09% target coverage, 3.64x average depth, and an efficiency of 29.3; record these as one statistics row per design (e.g., the Table 1 'Sepsis HUBDesign' entry) so naive tiling baselines can be compared directly.

## Related tools

- **HUBDesign** (Produces the probe set (Clustering, Assignment, Identification, Filtering, Selection) whose final FASTA is counted and characterized by these statistics) — https://github.com/zacherydickson/HUBDesign
- **BLASTn** (Used during the Filtering phase against blacklist databases (e.g., human GRCh38) so that only host-safe probes enter the counted set)
- **PROKKA** (Suggested annotation tool generating the annotated genome inputs from which total input nucleotides are derived)

## Examples

```
bin/Selection.pl [-options] -n maxBaits Tree Candidates > TiledBaits.fna
```

## Evaluation signals

- Probe count of the final FASTA equals 26,870 for the sepsis design reproducing task_003
- Computed target coverage matches 2.09% of all input nucleotides
- Average depth of coverage equals 3.64x over targeted bases
- Efficiency metric equals 29.3, matching the Table 1 'Sepsis HUBDesign' row
- Every counted probe passed Filtering against the blacklist database and was selected at 5 bp tiling density under the probe-count cap

## Limitations

- The published sepsis probe set was produced with an earlier HUBDesign version that had a cluster-partitioning flaw (genera treated independently rather than by observed sequence similarity), so current versions may not reproduce the exact 26,870-probe figures
- Efficiency and enrichment are dependent on the sequence properties of the targets; the level of enrichment will vary for each probe set produced
- Default --probe-count is a multiple of the number of input genomes, so counts are cap-dependent; --tiling-density set too high can make balancing impossible and yield unexpectedly small probe sets

## Evidence

- [results] HUBDesign produced a compact sepsis probe set of 26,870 probes targeting bacterial pathogens, covering 2.09% of all input nucleotides at an average depth of coverage of 3.64x.: [evidence span withheld — anchor status: not_found]
- [discussion] The HUBDesign pipeline was able to rapidly design a compact and efficient probe set covering almost every one of the targeted coronaviruses.: "The HUBDesign pipeline was able to rapidly design a compact and efficient probe set covering almost every one of the targeted coronaviruses."
- [readme] A fasta formatted set of probe sequences; A tab delimited file describing the source and position of the probe sequences: "A fasta formatted set of probe sequences; A tab delimited file describing the source and position of the probe sequences"
- [readme] By default the maximum number of probes will be a multiple of the number of input genomes: "The maximum size of the output probe set. By default the maximum number of probes will be a multiple of
the number of input genomes"
