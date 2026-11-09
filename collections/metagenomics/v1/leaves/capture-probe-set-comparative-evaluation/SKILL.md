---
name: capture-probe-set-comparative-evaluation
description: Use when multiple probe design strategies must be compared on identical
  target genomes (e.g., 56 RefSeq coronaviruses) and you need a quantitative efficiency
  metric — average distinct genomes hit per probe — to trade off probe count versus
  breadth.
when_to_use_negative:
- Input genomes are unknown or unannotatable — HUBDesign relies on annotated genomes,
  so probe sets cannot be designed or fairly benchmarked for them
- The goal is wet-lab capture validation (enrichment fold in pooled samples) rather
  than in silico probe-set comparison
- Only one design method or one probe set is of interest with no comparative baseline
  — there is nothing to benchmark against
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: HUBDesign
  role: Generates the homology-guided probe set being benchmarked; run with a guide
    tree from NCBI taxonomy, tiling density 1, and probe count 5000 on the 56-genome
    test set
  repo: https://github.com/zacherydickson/HUBDesign
- name: CATCH
  role: Produces the two comparison probe sets on the same genomes, once with strict
    hybridization parameters (exact matches only) and once with permissive parameters
    (≤18 mismatches, 15 bp exact island)
- name: BLASTn
  role: Maps each candidate probe set back against the 56 target genomes to compute
    per-probe genome hits for the efficiency metric
- name: PROKKA
  role: Annotates the raw target genomes (kingdom set to viruses) to create the HUBDesign-compatible
    input files named by taxon-id
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:9e3337b05efd75e419ab99133f0feb1ac89df8d750288e8e08f29049519ba143
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
  - CATCH
  - HUBDesign
  - PROKKA
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# capture-probe-set-comparative-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks hybridization-capture probe design methods (HUBDesign vs CATCH strict/permissive vs naive tiling) on the same target genome set, using probe-set size and an 'efficiency' metric — the average number of distinct target genomes each probe maps to — to judge compactness versus breadth.

## When to use

Use when multiple probe design strategies must be compared on identical target genomes (e.g., 56 RefSeq coronaviruses) and you need a quantitative efficiency metric — average distinct genomes hit per probe — to trade off probe count versus breadth.

## When NOT to use

- Input genomes are unknown or unannotatable — HUBDesign relies on annotated genomes, so probe sets cannot be designed or fairly benchmarked for them
- The goal is wet-lab capture validation (enrichment fold in pooled samples) rather than in silico probe-set comparison
- Only one design method or one probe set is of interest with no comparative baseline — there is nothing to benchmark against

## Inputs

- Target genome FASTA set (e.g., 56 RefSeq coronavirus genomes)
- PROKKA annotation GFF files for the target genomes (HUBDesign input)
- Guide tree derived from NCBI taxonomy lineages (HUBDesign --guide-tree input)
- CATCH parameter sets: strict (exact matches only) and permissive (≤18 mismatches in probe window, 15 bp exact-match island)
- Naive tiling specification: 75-mer probes at 15 bp spacing
- BLASTn database built from the target genomes

## Outputs

- Per-method probe sets in FASTA format
- BLASTn mapping results of each probe set against the target genomes
- Per-method probe counts and efficiency values (average distinct genomes hit per probe)
- Tab-delimited comparative benchmark table

## How to apply

Assemble the target set (56 RefSeq coronavirus genomes annotated with PROKKA, as in the HUBDesign test directory) and generate each candidate probe set on it: CATCH under a strict hybridization parameter set (exact matches only, no mismatches), CATCH under a permissive set (up to 18 mismatches within the probe window requiring a 15 bp exact-match island), naive tiling of 75-mers at 15 bp spacing without hybridization modeling, and the HUBDesign set (e.g., --guide-tree from NCBI taxonomy lineages, --tiling-density 1, --probe-count 5000). Map each probe set back to the 56 genomes with BLASTn and compute, per set, the number of probes and the efficiency: the average number of distinct genomes each probe hits. Report results in a tab-delimited table (e.g., CATCH strict: 3,846 probes, efficiency 1.13; CATCH permissive: 1,474 probes, efficiency 4.23; naive tiling: 21,267 probes; HUBDesign: 13,500 probes, efficiency 1.87). Rationale: CATCH strict is maximally specific but reuses probes little, permissive settings raise efficiency at the cost of mismatch tolerance, and naive tiling shows the size penalty of ignoring hybridization.

## Related tools

- **HUBDesign** (Generates the homology-guided probe set being benchmarked; run with a guide tree from NCBI taxonomy, tiling density 1, and probe count 5000 on the 56-genome test set) — https://github.com/zacherydickson/HUBDesign
- **CATCH** (Produces the two comparison probe sets on the same genomes, once with strict hybridization parameters (exact matches only) and once with permissive parameters (≤18 mismatches, 15 bp exact island))
- **BLASTn** (Maps each candidate probe set back against the 56 target genomes to compute per-probe genome hits for the efficiency metric)
- **PROKKA** (Annotates the raw target genomes (kingdom set to viruses) to create the HUBDesign-compatible input files named by taxon-id)

## Examples

```
bin/HUBDesign --guide-tree test/Guide.tree --tiling-density 1 --probe-count 5000 --output-dir test_res --verbose --keep test/GenomeInfo.tab 2>test.log
```

## Evaluation signals

- Efficiency values reproduce the reported pattern: CATCH strict low (~1.13), CATCH permissive highest (~4.23), HUBDesign intermediate (~1.87 with 13,500 probes), naive tiling largest probe set (21,267)
- Naive tiling probe count matches expectation from genome sizes: one 75-mer per 15 bp step across all 56 genomes with no deduplication
- All probe sets evaluated were generated against the identical target genome set (same 56 RefSeq coronaviruses), so counts are directly comparable
- Efficiency metric is computed consistently as the mean number of distinct genomes each probe hits in BLASTn mapping against all 56 genomes
- Benchmark rows are assembled into a tab-delimited table with probe count and efficiency per method

## Limitations

- Efficiency is target-sequence dependent: 'the performance of probes is dependent on the sequence properties of the targets, the level of enrichment will vary for each probe set produced'
- HUBDesign requires annotated genomes, so the comparison cannot extend to unannotatable targets; probes for some taxa (e.g., txid1503293, txid693998) are absent under the published version
- Naive tiling ignores hybridization modeling and thus inflates probe count, while CATCH strict under-reuses probes; neither extreme is directly practical
- CATCH permissive tolerates up to 18 mismatches (15 bp exact island), so its high efficiency partly reflects relaxed specificity rather than better design

## Evidence

- [readme] Genome files were created by running PROKKA on the raw genome files with the kingdom set to viruses, names of the files are the taxon-ids of the genomes: "Genome files were created by running PROKKA on the raw genome files with the kingdom set to viruses, names of the files are the taxon-ids of the genomes"
- [discussion] Comparable performance to CATCH: "The HUBDesign pipeline was able to rapidly design a compact and efficient probe set covering almost every one of the targeted coronaviruses."
- [methods] Guide tree provenance: "the lineages provided in NCBI's taxonomy for 56 refseq coronavirus genomes"
- [readme] Run command for reproducing the benchmarked HUBDesign output: "The output of this command can be compared to the output provided in the test directory"
