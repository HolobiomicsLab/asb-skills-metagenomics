---
name: homology-based-offtarget-probe-filtration
description: Use when a candidate bait-region table (CandidateBaitRegions.tab) from
  the identification stage must be purged of probes with homologous hits to host or
  contaminant reference sequences before final probe selection.
when_to_use_negative:
- The input is not yet a candidate bait-region table — e.g. raw FASTQ reads needing
  human-read filtration or deduplication, which are downstream analysis filters, not
  probe-set filtration.
- The target genomes are unknown or unannotatable, so no candidate probes could be
  identified and there is nothing to filter.
- You only need the final balanced probe set and have no off-target/contaminant sequences
  to screen against — in that case the Selection stage alone may suffice, though skipping
  filtration risks cross-reactivity.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: HUBDesign
  role: Pipeline whose Filtering (bin/Filtering.pl) and Selection (bin/Selection.pl)
    stages implement off-target removal and balanced probe selection
  repo: https://github.com/zacherydickson/HUBDesign
- name: BLASTn
  role: Screens candidate baits against blacklist databases to detect off-target homology
    (version 2.9.1+)
- name: sdust
  role: LCR masker that masks low-complexity intervals in remaining candidates before
    selection (version 1.0.0+)
- name: BioPerl
  role: Required Perl library dependency for running HUBDesign's Filtering and Selection
    scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:bf266ed01c310aa334c2c9ee44f4b5b547f64f8681fcd147518dea841187493a
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
  - HUBDesign
  - Perl
  - sdust
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# homology-based-offtarget-probe-filtration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A HUBDesign pipeline stage that removes candidate hybridization probes predicted to capture off-target sequences by BLASTn screening against blacklist databases, and masks low-complexity regions, before balanced probe selection. It is the step that converts SA_BOND-identified candidate bait regions into a specificity-checked candidate pool.

## When to use

Use when a candidate bait-region table (CandidateBaitRegions.tab) from the identification stage must be purged of probes with homologous hits to host or contaminant reference sequences before final probe selection.

## When NOT to use

- The input is not yet a candidate bait-region table — e.g. raw FASTQ reads needing human-read filtration or deduplication, which are downstream analysis filters, not probe-set filtration.
- The target genomes are unknown or unannotatable, so no candidate probes could be identified and there is nothing to filter.
- You only need the final balanced probe set and have no off-target/contaminant sequences to screen against — in that case the Selection stage alone may suffice, though skipping filtration risks cross-reactivity.

## Inputs

- Candidate bait regions table (CandidateBaitRegions.tab) from the Identification stage
- BLASTn blacklist sequence databases (e.g. GRCh38 human reference, NCBI non-redundant nucleotide database)
- LCR masker binary (sdust version 1.0.0+ or dustmasker version 1.0.0+, can be disabled)
- BLASTn version 2.9.1+
- Optionally, a guide tree and probe count (-n maxBaits) for the downstream Selection stage

## Outputs

- FilteredCandidates.tab — tab-delimited table of candidate bait regions with blacklist-cross-reactive probes removed
- TiledBaits.fna — FASTA file of selected probe sequences (post-filtration, post-selection)
- Tab-delimited report of probe source and position plus per-taxon probe counts

## How to apply

Take the candidate bait regions table produced by Identification.pl and run bin/Filtering.pl, supplying blacklist BLASTn databases (e.g. GRCh38 for human background, or NCBI's non-redundant nucleotide database for non-viral hits) via the --blast-db option or positional BlastDB arguments. Candidates with significant homology to blacklist sequences are discarded so the final probe set cannot capture off-target targets. Low-complexity intervals within surviving candidates are masked with an LCR masker — dustmasker or sdust are supported (masking can be disabled) — since low-complexity regions drive spurious cross-hybridization. As long as the tab-delimited format is preserved, the resulting FilteredCandidates.tab feeds directly into bin/Selection.pl, where iterative tiling to a minimum tiling-density target also balances the number of probes per input genome. The filtration and selection stages are separable on purpose so additional processing (e.g. filtering against NCBI nr for non-viral hits) can be inserted between them.

## Related tools

- **HUBDesign** (Pipeline whose Filtering (bin/Filtering.pl) and Selection (bin/Selection.pl) stages implement off-target removal and balanced probe selection) — https://github.com/zacherydickson/HUBDesign
- **BLASTn** (Screens candidate baits against blacklist databases to detect off-target homology (version 2.9.1+))
- **sdust** (LCR masker that masks low-complexity intervals in remaining candidates before selection (version 1.0.0+))
- **BioPerl** (Required Perl library dependency for running HUBDesign's Filtering and Selection scripts)

## Examples

```
Filtering.pl [-options] Candidates.tab [BlastDB_1 ...] > FilteredCandidates.tab
```

## Evaluation signals

- FilteredCandidates.tab is produced and preserves the input tab-delimited format so it can be passed directly to Selection.pl.
- The balance/selection report shows no blacklist-cross-reactive candidates remaining in the final probe set.
- TiledBaits.fna exists and contains at most -n maxBaits probes (e.g. at most 13,500 when run with -n 13500).
- Per-taxon probe counts in the report are balanced under the --tiling-density minimum target without producing an unexpectedly-small probe set.
- Default probe-count behavior (a multiple of the number of input genomes) is respected if --probe-count is not set.

## Limitations

- The pipeline ships without contaminant blacklist databases; the article notes that 'backlist databases of common contaminant sequences could be provided to HUBDesign during the filtration phase' as a future capability, so users must supply appropriate BLAST databases themselves.
- Filtration quality is only as good as the blacklist databases provided — off-targets absent from the BlastDB will not be removed.
- Setting --tiling-density too high can make it impossible to balance baits across organisms, causing unexpectedly-small probe sets even after successful filtration.
- HUBDesign relies on annotated genomes to cluster sequences; probes (and thus filtration) cannot be designed for unknown or unannotatable genomes.

## Evidence

- [intro] probes that would capture off-target sequences are removed. In the final step, a set of probes that balances the number of probes per input genome is selected: "probes that would capture off-target sequences are removed. In the final step, a set of probes that balances the number of probes per input genome is selected"
