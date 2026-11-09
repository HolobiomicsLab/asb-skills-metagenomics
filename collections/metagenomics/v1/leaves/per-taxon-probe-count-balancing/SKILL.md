---
name: per-taxon-probe-count-balancing
description: Use when building a hybridization-capture probe set from HUBDesign CandidateBaitRegions.tab
  and you need the final probe count capped and distributed fairly across all input
  taxa, e.g. tiling to 5x density with at most 13,500 baits.
when_to_use_negative:
- Input is unannotated or unannotatable genomes — HUBDesign cannot design probes and
  only close relatives can be targeted
- You only need to quantify specificity of an existing probe set (map probes back
  via BLASTn rather than re-run selection)
- You need intermediate-stage reprocessing only up to candidate identification (CandidateBaitRegions.tab)
  without final tiling
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HUBDesign
  role: Pipeline providing the Filtering and Selection stages that remove off-target
    candidates and select a probe count balanced per input genome
  repo: github.com/zacherydickson/HUBDesign
- name: BLASTn
  role: Aligns candidate baits against blacklist databases (version 2.9.1+) to remove
    off-target candidates before selection
- name: sdust
  role: LCR masker that masks low-complexity intervals in remaining candidates before
    selection
- name: BioPerl
  role: Required Perl library (1.7.10+) for the HUBDesign scripts
provenance:
  source_task_ids:
  - task_008
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:789b54b7a3473f1008561f30682bbbbebd36d9d0735527216af607a4eca99322
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/zacherydickson/HUBDesign
  tools:
  - BLASTn
  - BioPerl
  - HUBDesign
  - Perl
  - sdust
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# per-taxon-probe-count-balancing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Balanced probe selection in the HUBDesign pipeline: after filtering off-target candidates against blacklist BLAST databases, the Selection stage tiles remaining candidate bait regions while balancing the number of probes per input genome under a minimum tiling-density target.

## When to use

Use when building a hybridization-capture probe set from HUBDesign CandidateBaitRegions.tab and you need the final probe count capped and distributed fairly across all input taxa, e.g. tiling to 5x density with at most 13,500 baits.

## When NOT to use

- Input is unannotated or unannotatable genomes — HUBDesign cannot design probes and only close relatives can be targeted
- You only need to quantify specificity of an existing probe set (map probes back via BLASTn rather than re-run selection)
- You need intermediate-stage reprocessing only up to candidate identification (CandidateBaitRegions.tab) without final tiling

## Inputs

- Candidate bait regions table (CandidateBaitRegions.tab / FilteredCandidates.tab)
- Newick-format guide tree of the input taxa (Guide.tree)
- Blacklist BLASTn databases (e.g. GRCh38, NCBI non-redundant nucleotide for non-viral hits)
- Parameters: maxBaits (-n), --tiling-density, --probe-count, --probe-length

## Outputs

- TiledBaits.fna — FASTA file of selected probe sequences (at most maxBaits probes)
- Tab-delimited report of the source and position of each probe plus per-taxon probe counts

## How to apply

Run the Filtering stage (bin/Filtering.pl) on the candidate bait regions against blacklist BLASTn databases (e.g. GRCh38, discarding candidates with >=75% identity over >=30 bp, e-value <0.01) and mask low-complexity intervals with the sdust or dustmasker LCR masker. Then run the Selection stage (bin/Selection.pl) with the guide tree, the filtered candidates, and -n maxBaits to cap the output probe set; the default maximum is a multiple of the number of input genomes. Choose the --tiling-density value deliberately: higher minimum tiling density improves capture but can make balancing baits across organisms impossible, producing unexpectedly-small probe sets. Rationale: balancing probe counts per genome prevents abundant, probe-rich taxa from dominating the set while rare taxa remain coverable, exploiting hybridization to sequences up to 20% divergent for breadth.

## Related tools

- **HUBDesign** (Pipeline providing the Filtering and Selection stages that remove off-target candidates and select a probe count balanced per input genome) — github.com/zacherydickson/HUBDesign
- **BLASTn** (Aligns candidate baits against blacklist databases (version 2.9.1+) to remove off-target candidates before selection)
- **sdust** (LCR masker that masks low-complexity intervals in remaining candidates before selection)
- **BioPerl** (Required Perl library (1.7.10+) for the HUBDesign scripts)

## Examples

```
bin/Selection.pl --tiling-density 5 -n 13500 Guide.tree FilteredCandidates.tab > TiledBaits.fna
```

## Evaluation signals

- TiledBaits.fna exists and contains at most maxBaits probes (e.g. 13,500 as used for the coronavirus/sepsis-scale runs)
- Balance report shows per-taxon probe counts distributed under the minimum tiling-density target (e.g. 5x) with no taxon dominating
- No blacklist-cross-reactive candidates remain after Filtering (all discarded under the >=75% identity / >=30 bp / e-value <0.01 rule)
- Full-pipeline test reproduces the repository test output (e.g. 8 probes for each of the 23 gammacoronaviruses; zero probes for txid1503293 and txid693998)
- If the probe set is unexpectedly small, lower --tiling-density, since an unreachable tiling target prevents balancing across organisms

## Limitations

- Enrichment and probe count are target-sequence dependent and will vary for each probe set produced; the --tiling-density target set too high can cause unexpectedly-small probe sets
- By default the maximum number of probes is a multiple of the number of input genomes, constraining set size unless --probe-count is set
- The published sepsis probe set had a cluster-partitioning flaw (nominal taxonomy rather than observed sequence similarity) affecting which candidate probes were truly specific; filtering and selection cannot fix upstream assignment errors

## Evidence
