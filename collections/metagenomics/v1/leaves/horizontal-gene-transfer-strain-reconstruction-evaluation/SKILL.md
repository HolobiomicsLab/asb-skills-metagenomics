---
name: horizontal-gene-transfer-strain-reconstruction-evaluation
description: Use when comparing reconstruction accuracy (RA) of HGT/transferred-segment
  detection methods on WGSIM-simulated shotgun NGS data across sequencing coverages
  (e.g., 5X vs 30X), to test whether a graph-based method outperforms a breakpoint-only
  tool.
when_to_use_negative:
- Input is real (non-simulated) metagenomic data without ground-truth transferred
  segments, so RA cannot be computed against known truth
- Only one coverage level or only one tool's RA values are available, making ΔRA and
  its coverage trend undefined
- The goal is de novo HGT strain reconstruction itself rather than benchmarking reconstruction
  accuracy against LUMPY
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: LEMON
  role: Graph-based method whose reconstruction accuracy is evaluated; detects HGT
    breakpoints, identifies transferred segments, and reconstructs the inserted local
    strain
  repo: https://github.com/lichen2018/LEMON
- name: LUMPY
  role: Baseline third-party breakpoint caller used with default parameters to provide
    comparison RA values
- name: WGSIM
  role: Generates the simulated paired-end reads used to build the RA reproduction
    outputs at multiple coverage levels
- name: BWA
  role: Aligns simulated paired-end shotgun reads against reference genomes to enable
    junction-read detection
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:0892e7412921896e2e57d2dd787f9e43042ebcae780239fbfb8dcaa7cf18123e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - LEMON
  - LUMPY
  - WGSIM
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# horizontal-gene-transfer-strain-reconstruction-evaluation

## Summary

Quantitatively evaluates graph-based HGT strain reconstruction (LEMON) against a breakpoint-only baseline (LUMPY) by computing delta-RA = RA_LEMON − RA_LUMPY from WGSIM-simulated gut metagenomics reads at 5X and 30X coverage, mirroring Fig. 5a of the LEMON paper.

## When to use

Use when comparing reconstruction accuracy (RA) of HGT/transferred-segment detection methods on WGSIM-simulated shotgun NGS data across sequencing coverages (e.g., 5X vs 30X), to test whether a graph-based method outperforms a breakpoint-only tool.

## When NOT to use

- Input is real (non-simulated) metagenomic data without ground-truth transferred segments, so RA cannot be computed against known truth
- Only one coverage level or only one tool's RA values are available, making ΔRA and its coverage trend undefined
- The goal is de novo HGT strain reconstruction itself rather than benchmarking reconstruction accuracy against LUMPY

## Inputs

- Simulated Reconstruction Accuracy (RA) values for LEMON at 5X and 30X coverage from reproduction output files
- Simulated Reconstruction Accuracy (RA) values for LUMPY at 5X and 30X coverage from reproduction output files
- WGSIM-simulated paired-end shotgun reads (4 repetitions) aligned against the microbiome reference set S

## Outputs

- delta_ra_summary.csv containing ΔRA per coverage level (5X, 30X) and pass/fail verdicts for the positivity and trend criteria

## How to apply

Load the Reconstruction Accuracy (RA) values for LEMON and LUMPY from the upstream reproduction outputs, produced with WGSIM-simulated paired-end reads over 4 repetitions against the NCBI-derived microbiome reference set S (16,093 species, 1,246,881 scaffolds). For each coverage level compute ΔRA = RA_LEMON − RA_LUMPY, following the relation underlying Fig. 5a. Apply two criteria: (1) positivity — every ΔRA must be greater than 0, showing LEMON reaches higher RA than LUMPY at both 5X and 30X; (2) trend — ΔRA at 5X must exceed ΔRA at 30X, i.e. ΔRA decreases as coverage increases because LUMPY's RA improves with more data. Write per-coverage ΔRA values and pass/fail verdicts to delta_ra_summary.csv. The rationale is that LEMON reconstructs the full local strain (breakpoints plus transferred segments) while LUMPY only calls breakpoints, so the RA gap narrows at higher coverage.

## Related tools

- **LEMON** (Graph-based method whose reconstruction accuracy is evaluated; detects HGT breakpoints, identifies transferred segments, and reconstructs the inserted local strain) — https://github.com/lichen2018/LEMON
- **LUMPY** (Baseline third-party breakpoint caller used with default parameters to provide comparison RA values)
- **WGSIM** (Generates the simulated paired-end reads used to build the RA reproduction outputs at multiple coverage levels)
- **BWA** (Aligns simulated paired-end shotgun reads against reference genomes to enable junction-read detection)

## Examples

```
python LEMON/Scripts/get_raw_bkp.py -r reference.fa -u unique_reads.bam -o raw_breakpoints.txt -t 4
```

## Evaluation signals

- All computed ΔRA values (both 5X and 30X) are strictly greater than 0
- ΔRA at 5X exceeds ΔRA at 30X, reproducing the reported decreasing trend of Fig. 5a
- delta_ra_summary.csv contains exactly one ΔRA row per coverage level plus pass/fail verdicts for both criteria
- Underlying RA values derive from 4 WGSIM simulation repetitions, not a single run

## Limitations

- Evaluation relies on WGSIM simulations on reference sequences of the microbiome, not real HMP IBD metagenomic data
- The trend criterion assumes LUMPY's RA improves with coverage; other baselines may not show the same coverage-dependence
- RA on reference genomes does not capture complex structural variation present in real transferred segments

## Evidence

- [abstract] The simulated results indicate that the method can identify transferred segments accurately on reference sequences of the microbiome.: "The simulated results indicate that the method can identify transferred segments accurately on reference sequences of the microbiome."
- [methods] All true local strains were used to generated 20X paired-end reads with WGSIM [28] as an input of LEMON.: "All true local strains were used to generated 20X paired-end reads with WGSIM [28] as an input of LEMON."
- [readme] LEMON takes use of existing shotgun NGS datasets to detect HGT breakpoints, identify the transferred genome segments, and reconstructs the inserted local strain.: "It is a software takes use of existing shotgun NGS datasets to detect HGT breakpoints, identify the transferred genome segments, and reconstructs the inserted local strain."
- [methods] Most third-party tools used in this article are set default parameters, including BWA, LUMPY [25], iRep [26], and STAR-Fusion [27].: "Most third-party tools used in this article are set default parameters, including BWA, LUMPY [25], iRep [26], and STAR-Fusion [27]."
