---
name: viral-genome-sequence-homology-mapping
description: Use when comparing oligonucleotide probe sets designed across multiple
  related viral genomes (e.g., 56 RefSeq coronaviruses) and you need a quantitative
  efficiency metric — mean distinct-genome hits per probe — to rank competing design
  strategies.
when_to_use_negative:
- Input genomes are unknown or unannotatable — HUBDesign relies on annotated genomes
  to efficiently cluster sequences, so probes cannot be designed for such targets,
  only close relatives
- The task is wet-lab capture validation (enrichment on real or spiked samples) rather
  than in-silico probe-set benchmarking; this skill computes design metrics only
- A single near-identical genome target where homology-aware hierarchical design offers
  no advantage over simple tiling
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: HUBDesign
  role: Source of the homology-based probe set under evaluation; designed from PROKKA-annotated
    genomes and an NCBI-taxonomy guide tree
  repo: https://github.com/zacherydickson/HUBDesign
- name: CATCH
  role: Produces the two benchmark competitor probe sets with strict (exact-match)
    and permissive (≤18 mismatches, 15 bp exact island) hybridization parameter sets
- name: BLASTn
  role: Maps each probe set back against the 56 target genomes to count distinct-genome
    hits per probe for the efficiency metric
- name: PROKKA
  role: Annotates the raw coronavirus genomes (kingdom set to viruses) to generate
    the HUBDesign input GFF files
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:769e8fc024c8328cee0a47a97af07a2b6a9053954885b1865733ebd448d4548c
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# viral-genome-sequence-homology-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks hybridization-capture probe sets for coronavirus genomes by mapping each probe set back to the target genomes with BLASTn and computing probe-set efficiency (the average number of distinct genomes each probe maps to), enabling head-to-head comparison of HUBDesign, CATCH (strict and permissive), and naive tiling.

## When to use

Use when comparing oligonucleotide probe sets designed across multiple related viral genomes (e.g., 56 RefSeq coronaviruses) and you need a quantitative efficiency metric — mean distinct-genome hits per probe — to rank competing design strategies.

## When NOT to use

- Input genomes are unknown or unannotatable — HUBDesign relies on annotated genomes to efficiently cluster sequences, so probes cannot be designed for such targets, only close relatives
- The task is wet-lab capture validation (enrichment on real or spiked samples) rather than in-silico probe-set benchmarking; this skill computes design metrics only
- A single near-identical genome target where homology-aware hierarchical design offers no advantage over simple tiling

## Inputs

- 56 RefSeq coronavirus genomes, PROKKA-annotated (GFF + FASTA, taxon-id-named files as in the HUBDesign test/ directory)
- GenomeInfo.tab (tab-delimited paths-to-annotated-genome plus taxon-id)
- Guide.tree (Newick tree from NCBI taxonomy lineages for the 56 genomes)
- Probe sets to benchmark: CATCH strict set, CATCH permissive set, naive 75-mer tiling at 15 bp spacing (FASTA), HUBDesign TiledBaits.fna
- BLASTn database built from the 56 target genomes

## Outputs

- Per-probe-set probe counts and BLASTn hits against all 56 genomes
- Efficiency metric per probe set: average number of distinct genomes each probe maps to (e.g., HUBDesign 1.87; CATCH strict 1.13; CATCH permissive 4.23)
- Tab-delimited benchmark table comparing probe counts and efficiencies across CATCH strict, CATCH permissive, naive tiling, and HUBDesign

## How to apply

Assemble the annotated target genomes (e.g., 56 RefSeq coronavirus genomes annotated with PROKKA, as in the HUBDesign test directory) and generate the probe sets to compare: CATCH strict (exact matches only), CATCH permissive (up to 18 mismatches in the probe window with a 15 bp exact-match island), naive tiling of 75-mers at 15 bp spacing, and HUBDesign output (e.g., 75-mers run with --tiling-density 1 and a probe-count cap). Map every probe set back against the full 56-genome target set with BLASTn, then compute per set: number of probes and efficiency = average number of distinct genomes hit per probe. Report the rows side by side (CATCH strict: 3,846 probes, efficiency 1.13; CATCH permissive: 1,474 probes, 4.23; naive tiling: 21,267 probes; HUBDesign: 13,500 probes, 1.87); the metric exposes the trade-off between probe-set compactness and breadth (permissive CATCH is efficient per probe but small; naive tiling is exhaustive but redundant). HUBDesign's rationale is that hybridization to sequences up to 20% divergent is possible, so homology-clustered representative sequences at taxonomic nodes yield a compact set that still captures known and related viruses.

## Related tools

- **HUBDesign** (Source of the homology-based probe set under evaluation; designed from PROKKA-annotated genomes and an NCBI-taxonomy guide tree) — https://github.com/zacherydickson/HUBDesign
- **CATCH** (Produces the two benchmark competitor probe sets with strict (exact-match) and permissive (≤18 mismatches, 15 bp exact island) hybridization parameter sets)
- **BLASTn** (Maps each probe set back against the 56 target genomes to count distinct-genome hits per probe for the efficiency metric)
- **PROKKA** (Annotates the raw coronavirus genomes (kingdom set to viruses) to generate the HUBDesign input GFF files)

## Examples

```
bin/HUBDesign --guide-tree test/Guide.tree --tiling-density 1 --probe-count 5000 --output-dir test_res --verbose --keep test/GenomeInfo.tab 2>test.log
```

## Evaluation signals

- BLASTn mapping of the HUBDesign test output yields ~13,500 probes with efficiency ≈1.87 distinct genomes per probe on the 56-genome set
- CATCH strict row reproduces ≈3,846 probes at efficiency 1.13 and CATCH permissive ≈1,474 probes at 4.23; naive tiling yields ≈21,267 probes
- HUBDesign output probe count does not exceed the --probe-count cap and defaults to a multiple of the number of input genomes
- Output files match the HUBDesign test directory: probe FASTA, tab-delimited probe source/position table, cluster composition and cluster FASTA files, and a Newick tree
- Every benchmarked probe is a 75-mer and the naive-tiling set is spaced at 15 bp intervals as specified

## Limitations

- Enrichment/performance is target-sequence dependent and will vary for each probe set produced; there is no general guarantee
- The current HUBDesign version constructs a new cluster-assignment tree rather than using the published guide tree — a version discrepancy with what was validated (e.g., no probes exist for txid1503293 and txid693998 under the published version)
- SA_BOND identification scales poorly: memory grows linearly with genome size and organism number, but time requirements grow much more quickly with diversity

## Evidence
