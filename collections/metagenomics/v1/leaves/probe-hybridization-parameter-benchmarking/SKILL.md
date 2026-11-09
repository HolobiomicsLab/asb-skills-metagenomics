---
name: probe-hybridization-parameter-benchmarking
description: Use when comparing probe-design pipelines or hybridization parameter
  sets on a fixed multi-genome target set and you need a quantitative metric (probes
  per set, average distinct genomes hit per probe) to judge compactness vs coverage
  trade-offs.
when_to_use_negative:
- You only have a single genome or a set of nearly identical strains — hybridization-parameter
  benchmarking across diverse genomes is not informative there.
- Targets have unknown or unannotatable genomes — HUBDesign relies on annotated genomes
  to cluster sequences, so annotated-genome-dependent designs (including its efficiency
  comparison) cannot be produced.
- You need experimentally validated capture performance (fold enrichment) rather than
  in silico probe-set comparison — this skill only benchmarks computational probe
  sets.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: HUBDesign
  role: Homology-based hierarchical probe design pipeline whose output probe set is
    one benchmark row; run end-to-end on annotated coronavirus genomes with a Guide.tree
  repo: https://github.com/zacherydickson/HUBDesign
- name: CATCH
  role: Produces the two comparison probe sets under strict (exact match) and permissive
    (up to 18 mismatches with a 15 bp exact-match island) hybridization parameter
    sets
- name: BLASTn
  role: Maps each probe set back against the 56 target genomes to compute probe counts
    and efficiency (average distinct genomes hit per probe)
- name: PROKKA
  role: Annotates the raw genome files before HUBDesign input; HUBDesign takes annotated
    genomes as input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:5e6412d97b783a84851b8e00b0d835c963b43dd93e52a2c0f9a1e9e48f821bd7
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
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# probe-hybridization-parameter-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark hybridization-based probe design (CATCH strict/permissive, HUBDesign) against naive tiling on the same 56 RefSeq coronavirus genomes, using probe-set size and 'efficiency' (average number of distinct genomes each probe maps to) as comparison metrics.

## When to use

Use when comparing probe-design pipelines or hybridization parameter sets on a fixed multi-genome target set and you need a quantitative metric (probes per set, average distinct genomes hit per probe) to judge compactness vs coverage trade-offs.

## When NOT to use

- You only have a single genome or a set of nearly identical strains — hybridization-parameter benchmarking across diverse genomes is not informative there.
- Targets have unknown or unannotatable genomes — HUBDesign relies on annotated genomes to cluster sequences, so annotated-genome-dependent designs (including its efficiency comparison) cannot be produced.
- You need experimentally validated capture performance (fold enrichment) rather than in silico probe-set comparison — this skill only benchmarks computational probe sets.

## Inputs

- PROKKA-annotated target genomes (FASTA + GFF; 56 RefSeq coronavirus genomes as provided in the HUBDesign test directory)
- CATCH strict-parameter probe set (exact matches only, no mismatches allowed)
- CATCH permissive-parameter probe set (up to 18 mismatches within the probe window, requiring a 15 bp exact-match island)
- Naive tiling probe set (75-mers tiled at 15 bp spacing, no hybridization modeling)
- HUBDesign probe set (FASTA of probe sequences with a Guide.tree from NCBI taxonomy)

## Outputs

- Per-probe-set BLASTn mapping results against the 56 target genomes
- Efficiency metric per probe set (average number of distinct genomes each probe maps to)
- Tab-delimited benchmark table listing probe counts and efficiencies (e.g., CATCH strict: 3,846 probes, efficiency 1.13; CATCH permissive: 1,474 probes, 4.23; naive tiling: 21,267 probes; HUBDesign: 13,500 probes, 1.87)

## How to apply

Assemble the target genomes (PROKKA-annotated RefSeq coronavirus genomes, 56 in the HUBDesign test directory) and generate multiple probe sets: run CATCH under a strict parameter set (exact matches only, no mismatches) and a permissive set (up to 18 mismatches within the probe window with a required 15 bp exact-match island), and tile 75-mers every 15 bp across each genome as a naive baseline. Design the HUBDesign probe set with the pipeline (clustering, assignment, identification, filtering, selection; e.g., --tiling-density 1, --probe-count as desired) using a Guide.tree derived from NCBI taxonomy. Map each probe set back against the target genomes with BLASTn and compute, per set, the probe count and the efficiency — the average number of distinct genomes each probe hits. Assemble results into a tab-delimited benchmark table (e.g., CATCH strict 3,846 probes @ 1.13; CATCH permissive 1,474 probes @ 4.23; naive tiling 21,267 probes; HUBDesign 13,500 probes @ 1.87) and interpret efficiency against probe-set compactness.

## Related tools

- **HUBDesign** (Homology-based hierarchical probe design pipeline whose output probe set is one benchmark row; run end-to-end on annotated coronavirus genomes with a Guide.tree) — https://github.com/zacherydickson/HUBDesign
- **CATCH** (Produces the two comparison probe sets under strict (exact match) and permissive (up to 18 mismatches with a 15 bp exact-match island) hybridization parameter sets)
- **BLASTn** (Maps each probe set back against the 56 target genomes to compute probe counts and efficiency (average distinct genomes hit per probe))
- **PROKKA** (Annotates the raw genome files before HUBDesign input; HUBDesign takes annotated genomes as input)

## Examples

```
bin/HUBDesign --guide-tree test/Guide.tree --tiling-density 1 --probe-count 5000 --output-dir test_res --verbose --keep test/GenomeInfo.tab 2>test.log
```

## Evaluation signals

- Benchmark table reproduces the reported counts within tolerance: CATCH strict 3,846 probes (efficiency 1.13), CATCH permissive 1,474 probes (4.23), naive tiling 21,267 probes, HUBDesign 13,500 probes (1.87).
- BLASTn mapping is run for every probe set against the identical 56-genome target database, so efficiency values are directly comparable across rows.
- Naive tiling probe count is consistent with 15 bp spacing of 75-mers across the combined genome lengths (largest set by far).
- Each probe in every set is 75 nt (matching the probe-length convention) and maps to at least the genome it was drawn from.
- Output is a tab-delimited table with one row per design (method, parameter set, probe count, efficiency).

## Limitations

- HUBDesign relies on annotated genomes to efficiently cluster sequences; probes cannot be designed for targets with unknown or unannotatable genomes.
- In silico efficiency does not guarantee capture performance — the level of enrichment is dependent on the sequence properties of the targets and will vary for each probe set produced.
- SA_BOND (HUBDesign's candidate-identification step) time requirements grow quickly with genome size and number of organisms, so scaling the benchmark to much larger or more diverse target sets is costly.
- The published sepsis probe set had a cluster-partitioning flaw (nominal taxonomy rather than observed sequence similarity); a version discrepancy exists between the validated HUBDesign version and the current version that constructs its own cluster-assignment tree.

## Evidence

- [other] the efficiency of the HUBDesign coronavirus probe set ... average number of distinct genomes each probe maps to: [evidence span withheld — anchor status: not_found]
