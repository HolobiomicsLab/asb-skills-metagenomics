---
name: probe-cross-reactivity-efficiency-calculation
description: Use when comparing hybrid-capture probe design methods on the same target
  genome set and you need a quantitative cross-reactivity/coverage metric, i.e., the
  mean number of distinct target genomes hit per probe.
when_to_use_negative:
- The genome targets are unknown or unannotatable — HUBDesign relies on annotated
  genomes to cluster sequences, so probes cannot be designed for such targets (only
  close relatives)
- You need empirical wet-lab enrichment performance rather than in silico cross-reactivity
  — the paper notes enrichment level is target-sequence dependent and will vary for
  each probe set produced
- The target set is a single organism with no diversity to capture — the efficiency
  metric (distinct genomes hit per probe) is uninformative
tools:
- name: HUBDesign
  role: Generates the compact homology-based probe set whose efficiency is benchmarked
    against CATCH and naive tiling
  repo: github.com/zacherydickson/HUBDesign
- name: CATCH
  role: Produces comparison probe sets under strict (exact-match) and permissive (up
    to 18 mismatches, 15 bp exact-match island) hybridization parameter sets
- name: BLASTn
  role: Maps each probe set back against the 56 target genomes to count distinct genome
    hits per probe for the efficiency metric
- name: PROKKA
  role: Annotates the input genomes (kingdom=viruses) so they can be consumed by HUBDesign
    for probe design
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:dee20f5fed550720617cc60b60320d7ec90812ec8383e2db816735e561b2f25f
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
  - CATCH
  - HUBDesign
  - PROKKA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# probe-cross-reactivity-efficiency-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark metric that captures probe-set cross-reactivity: map each designed probe set (HUBDesign, CATCH strict/permissive, naive tiling) back to the target genomes with BLASTn and compute the average number of distinct genomes each probe hits ('efficiency'). It quantifies how compactly and broadly a probe set covers diverse targets on a shared genome set.

## When to use

Use when comparing hybrid-capture probe design methods on the same target genome set and you need a quantitative cross-reactivity/coverage metric, i.e., the mean number of distinct target genomes hit per probe.

## When NOT to use

- The genome targets are unknown or unannotatable — HUBDesign relies on annotated genomes to cluster sequences, so probes cannot be designed for such targets (only close relatives)
- You need empirical wet-lab enrichment performance rather than in silico cross-reactivity — the paper notes enrichment level is target-sequence dependent and will vary for each probe set produced
- The target set is a single organism with no diversity to capture — the efficiency metric (distinct genomes hit per probe) is uninformative

## Inputs

- 56 RefSeq coronavirus genomes, PROKKA-annotated (GFF files and taxon-id mapping via GenomeInfo.tab)
- Guide.tree (Newick) generated from NCBI taxonomy lineages for the 56 coronavirus genomes
- CATCH probe sets generated with strict (exact match) and permissive (up to 18 mismatches, 15 bp exact-match island) parameter sets
- Naive tiling probe set: 75-mers tiled at 15 bp spacing across each genome
- BLASTn database built from the 56 target genomes

## Outputs

- Per-probe-set probe count (e.g., HUBDesign 13,500; CATCH strict 3,846; CATCH permissive 1,474; naive tiling 21,267)
- Efficiency value per probe set: average number of distinct genomes hit per probe (e.g., HUBDesign 1.87; CATCH strict 1.13; CATCH permissive 4.23)
- Tab-delimited benchmark table comparing probe counts and efficiencies across methods

## How to apply

Assemble the target genome set — here the 56 RefSeq coronavirus genomes, PROKKA-annotated as provided in the HUBDesign test directory — as the reference for all comparisons. Generate each candidate probe set under identical target conditions: CATCH with strict hybridization (exact matches only, no mismatches), CATCH permissive (up to 18 mismatches in the probe window requiring a 15 bp exact-match island), naive tiling of 75-mers at 15 bp spacing without hybridization modeling, and the HUBDesign probe set (run via bin/HUBDesign on the PROKKA-annotated genomes with the NCBI-taxonomy-derived Guide.tree). Map each probe set back against the 56 genomes with BLASTn and, per probe set, record the number of probes and the average number of distinct genomes each probe hits — this is the 'efficiency' metric. Decision point: a lower probe count with reasonable efficiency (e.g., HUBDesign's 13,500 probes at efficiency 1.87 vs CATCH strict's 3,846 probes at 1.13 and CATCH permissive's 1,474 at 4.23) indicates a compact set that still captures multiple related genomes per probe; assemble the results into a tab-delimited benchmark table.

## Related tools

- **HUBDesign** (Generates the compact homology-based probe set whose efficiency is benchmarked against CATCH and naive tiling) — github.com/zacherydickson/HUBDesign
- **CATCH** (Produces comparison probe sets under strict (exact-match) and permissive (up to 18 mismatches, 15 bp exact-match island) hybridization parameter sets)
- **BLASTn** (Maps each probe set back against the 56 target genomes to count distinct genome hits per probe for the efficiency metric)
- **PROKKA** (Annotates the input genomes (kingdom=viruses) so they can be consumed by HUBDesign for probe design)

## Examples

```
bin/HUBDesign --guide-tree test/Guide.tree --tiling-density 1 --probe-count 5000 --output-dir test_res --verbose --keep test/GenomeInfo.tab 2>test.log
```

## Evaluation signals

- Published benchmark values are reproduced: HUBDesign 13,500 probes at efficiency 1.87; CATCH strict 3,846 probes at 1.13; CATCH permissive 1,474 probes at 4.23; naive tiling 21,267 probes
- Efficiency is computed as the per-probe-set mean over BLASTn hits of the number of distinct genomes each probe hits, not a per-probe hit count
- Naive tiling probe count is checkable arithmetic: 75-mers tiled at 15 bp spacing over the 56 genomes must yield 21,267 probes
- All methods must be evaluated against the identical 56-genome PROKKA-annotated target set so counts are comparable
- The benchmark output is a tab-delimited table with one row per method carrying probe count and efficiency columns

## Limitations

- CATCH permissive achieves higher nominal efficiency (4.23) but with far fewer probes (1,474); efficiency alone does not capture tiling density or coverage completeness
- Enrichment/performance of the resulting probe set is dependent on the sequence properties of the targets and will vary for each probe set produced
- HUBDesign relies on annotated genomes; probes cannot be designed for unknown or unannotatable targets
- SA_BOND, the candidate-identification step in HUBDesign, is the most memory-intensive step and its time requirements grow quickly with genome size and organism number

## Evidence
