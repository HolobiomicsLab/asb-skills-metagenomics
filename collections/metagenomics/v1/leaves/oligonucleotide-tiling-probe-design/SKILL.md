---
name: oligonucleotide-tiling-probe-design
description: Use when you must design targeted-capture baits across many related genomes
  (e.g., 56 RefSeq coronaviruses) and need to compare tiling-only, hybridization-modeled
  (CATCH), and homology-hierarchy (HUBDesign) strategies by efficiency metric.
when_to_use_negative:
- Target genomes are unknown or unannotatable — HUBDesign relies on annotated genomes
  to cluster sequences and cannot design probes for them
- You need validated wet-lab capture performance, not in silico benchmarking — the
  article notes enrichment is target-sequence dependent and validation used only artificial
  pooled samples
- A single-genome or single-locus assay is needed — efficiency gains come from multi-genome/multi-taxonomic-level
  design, not single-target capture
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0797
tools:
- name: HUBDesign
  role: Homology-aware probe design pipeline producing the benchmarked balanced probe
    set across taxonomic levels
  repo: https://github.com/zacherydickson/HUBDesign
- name: CATCH
  role: Generates comparison probe sets under strict (no mismatches) and permissive
    (up to 18 mismatches, 15 bp exact-match island) hybridization parameter sets
- name: BLASTn
  role: Maps each probe set back against the 56 target genomes to compute per-probe-set
    efficiency
- name: PROKKA
  role: Annotates the raw target genomes (kingdom=viruses) to produce HUBDesign input
    GFF files
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:def604064c91f26ac09dac29beea27c92717e3bebc96456dc0548a22932d8eb0
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# oligonucleotide-tiling-probe-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Designs and benchmarks hybridization-capture oligonucleotide probe sets (75-mers tiled at 15 bp spacing, CATCH strict/permissive, HUBDesign) against a multi-genome target set, scoring each set by probe count and capture efficiency (average number of distinct genomes each probe maps to).

## When to use

Use when you must design targeted-capture baits across many related genomes (e.g., 56 RefSeq coronaviruses) and need to compare tiling-only, hybridization-modeled (CATCH), and homology-hierarchy (HUBDesign) strategies by efficiency metric.

## When NOT to use

- Target genomes are unknown or unannotatable — HUBDesign relies on annotated genomes to cluster sequences and cannot design probes for them
- You need validated wet-lab capture performance, not in silico benchmarking — the article notes enrichment is target-sequence dependent and validation used only artificial pooled samples
- A single-genome or single-locus assay is needed — efficiency gains come from multi-genome/multi-taxonomic-level design, not single-target capture

## Inputs

- 56 RefSeq coronavirus genomes annotated with PROKKA (kingdom=viruses), filenames = taxon-ids
- GenomeInfo.tab: tab-delimited file of annotated-genome paths and taxon-ids
- Guide.tree: newick tree from NCBI taxonomy lineages of the input genomes
- Blacklist BLAST databases (e.g., NCBI non-redundant nucleotide database filtered for non-viral hits)

## Outputs

- FASTA probe sets from CATCH (strict and permissive), naive 75-mer tiling at 15 bp spacing, and HUBDesign
- Tab-delimited benchmark table of probe counts and efficiency (CATCH strict: 3,846 probes, eff 1.13; CATCH permissive: 1,474 probes, eff 4.23; naive tiling: 21,267 probes; HUBDesign: 13,500 probes, eff 1.87)
- BLASTn per-probe hit lists against the 56-genome target set
- HUBDesign outputs: probe FASTA, probe source/position table, gene-cluster composition table, cluster FASTA, newick tree

## How to apply

Annotate each target genome with PROKKA (kingdom=viruses) and assemble a GenomeInfo.tab of annotated-genome paths plus taxon-ids, plus a guide tree from NCBI taxonomy lineages. Generate comparison probe sets: CATCH with strict parameters (exact matches only, no mismatches), CATCH with permissive parameters (up to 18 mismatches within the probe window requiring a 15 bp exact-match island), and a naive set of 75-mers tiled at 15 bp spacing with no hybridization modeling. Run HUBDesign (Clustering, Assignment, Identification, Filtering, Selection; r2t-divergence controls cluster divergence) to produce a balanced, homology-aware probe set. Map every probe set back to all target genomes with BLASTn and compute probe count and efficiency = average distinct genomes hit per probe; on the coronavirus benchmark this yielded CATCH strict 3,846 probes @ 1.13, CATCH permissive 1,474 probes @ 4.23, naive tiling 21,267 probes, and HUBDesign 13,500 probes @ 1.87. Assemble results into a tab-delimited benchmark table.

## Related tools

- **HUBDesign** (Homology-aware probe design pipeline producing the benchmarked balanced probe set across taxonomic levels) — https://github.com/zacherydickson/HUBDesign
- **CATCH** (Generates comparison probe sets under strict (no mismatches) and permissive (up to 18 mismatches, 15 bp exact-match island) hybridization parameter sets)
- **BLASTn** (Maps each probe set back against the 56 target genomes to compute per-probe-set efficiency)
- **PROKKA** (Annotates the raw target genomes (kingdom=viruses) to produce HUBDesign input GFF files)

## Examples

```
bin/HUBDesign --guide-tree test/Guide.tree --tiling-density 1 --probe-count 5000 --output-dir test_res --verbose --keep test/GenomeInfo.tab 2>test.log
```

## Evaluation signals

- CATCH strict probe count is near 3,846 with efficiency ~1.13 (probes hit almost exactly one genome each on the 56-genome coronavirus set)
- Naive tiling yields ~21,267 probes at 15 bp spacing — an order of magnitude more probes than modeled approaches
- Per-probe efficiency (BLASTn-derived average distinct genomes hit per probe) falls in the ~1–5 range for modeled sets, with CATCH permissive highest (~4.23) and HUBDesign intermediate (~1.87 with 13,500 probes)
- HUBDesign output reproduces the repository test-directory files when run with --guide-tree test/Guide.tree --tiling-density 1 --probe-count 5000
- Benchmark table rows are complete and tab-delimited, with one row per strategy (CATCH strict, CATCH permissive, naive tiling, HUBDesign)

## Limitations

- HUBDesign requires annotated genomes; probes cannot be designed for unknown or unannotatable targets, only close relatives
- The sepsis probe set built with an earlier HUBDesign version had a cluster-partitioning flaw (nominal taxonomy rather than observed sequence similarity), letting highly divergent probes (e.g., Streptococcus spp.) into the set
- SA_BOND candidate identification is memory-intensive and time requirements grow quickly with genome size and organism number
- No contaminant backlist databases were provided; backlisting common contaminants during filtration is suggested future work
- Probe efficiency is benchmark-only — in-silico efficiency does not guarantee enrichment, which depends on target sequence properties

## Evidence
