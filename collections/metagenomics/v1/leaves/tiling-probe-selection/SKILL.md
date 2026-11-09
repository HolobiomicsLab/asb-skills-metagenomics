---
name: tiling-probe-selection
description: Use when annotated genomes (e.g., PROKKA GFFs) and a guide tree are available
  and you need a compact, balanced hybridization-capture probe set that captures known
  and related (up to ~20% divergent) sequences across multiple taxa.
when_to_use_negative:
- 'Targets have unknown or unannotatable genomes: HUBDesign relies on annotated genomes
  to cluster sequences, so probes cannot be designed for unknown targets (only close
  relatives can be targeted).'
- 'Setting --tiling-density higher than the data can support: an unrealistically high
  target can make it impossible to balance baits across organisms, causing unexpectedly
  small probe sets.'
- Single-organism, known-sequence detection tasks better served by PCR/multiplexed
  primer design rather than a taxonomically hierarchical capture probe set.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: HUBDesign
  role: End-to-end probe design pipeline (Clustering, Assignment, Identification,
    Filtering, Selection) producing the balanced tiled probe set
  repo: https://github.com/zacherydickson/HUBDesign
- name: SA_BOND
  role: Identification step that finds probes unique to each pseudo-genome; the most
    memory-intensive step
- name: BLASTn
  role: Filtering candidates against blacklist databases (GRCh38, nr nt) to remove
    off-target probes
- name: sdust
  role: 'LCR masker applied to remaining candidates before selection (alternative:
    dustmasker)'
- name: PROKKA
  role: Suggested tool to annotate input genomes before clustering
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:92bfb9e8c521b7ea4adf72c117a407433d1b6820863ca22fe5fb2b69cbcd13b7
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
  - SA_BOND
  - dustmasker
  - sdust
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# tiling-probe-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

HUBDesign's post-identification pipeline for hybridization-capture probe design: candidate bait regions identified by SA_BOND on pseudo-genomes are filtered against blacklist BLAST databases and low-complexity sequence, then a balanced probe set is selected by iterative tiling-density balancing across input genomes.

## When to use

Use when annotated genomes (e.g., PROKKA GFFs) and a guide tree are available and you need a compact, balanced hybridization-capture probe set that captures known and related (up to ~20% divergent) sequences across multiple taxa.

## When NOT to use

- Targets have unknown or unannotatable genomes: HUBDesign relies on annotated genomes to cluster sequences, so probes cannot be designed for unknown targets (only close relatives can be targeted).
- Setting --tiling-density higher than the data can support: an unrealistically high target can make it impossible to balance baits across organisms, causing unexpectedly small probe sets.
- Single-organism, known-sequence detection tasks better served by PCR/multiplexed primer design rather than a taxonomically hierarchical capture probe set.

## Inputs

- GenomeInfo.tab: tab-delimited file of paths to annotated genomes (GFF from PROKKA suggested) and their taxon IDs
- Guide.tree: Newick-format tree of the input genomes used to guide clustering/assignment hierarchy
- BLAST blacklist databases (e.g., GRCh38 human genome, NCBI non-redundant nucleotide database)
- Annotated genome GFF files (PROKKA output)

## Outputs

- TiledBaits.fna: FASTA-formatted set of final probe sequences (e.g., 75 bp sub-probes cut from 100 bp bait regions)
- Tab-delimited file describing the source and position of each probe sequence plus per-taxon probe counts
- Tab-delimited file detailing the composition of gene clusters
- FASTA-formatted set of cluster (representative) sequences
- Newick-formatted tree describing the relationship between taxa

## How to apply

Run bin/HUBDesign GenomeInfo.tab (tab-delimited file mapping annotated-genome paths to taxon IDs) with optional --guide-tree, --blast-db blacklist databases, --probe-count, --probe-length, --r2t-divergence, --penetrance, and --tiling-density parameters. The pipeline clusters genes (via MAFFT/MUSCLE/CLUSTAL OMEGA alignments, collapsing sequences at most 15% divergent into representative sequences), assigns representatives to the lowest common ancestor node, concatenates node representatives into pseudo-genomes, and runs SA_BOND to identify up to 100 candidate 100 bp bait regions per pseudo-genome. The Filtering stage (Filtering.pl) discards candidates with BLASTn hits against supplied blacklist databases (e.g., GRCh38) at >=75% identity over >=30 bp with e-value <0.01, then masks low-complexity intervals with sdust or dustmasker. The Selection stage (Selection.pl) performs iterative tiling-density balancing to a minimum tiling density (e.g., 5 bp) while balancing the number of probes per input genome, capped at the desired probe count (e.g., -n 13500).

## Related tools

- **HUBDesign** (End-to-end probe design pipeline (Clustering, Assignment, Identification, Filtering, Selection) producing the balanced tiled probe set) — https://github.com/zacherydickson/HUBDesign
- **SA_BOND** (Identification step that finds probes unique to each pseudo-genome; the most memory-intensive step)
- **BLASTn** (Filtering candidates against blacklist databases (GRCh38, nr nt) to remove off-target probes)
- **sdust** (LCR masker applied to remaining candidates before selection (alternative: dustmasker))
- **PROKKA** (Suggested tool to annotate input genomes before clustering)

## Examples

```
bin/HUBDesign --guide-tree test/Guide.tree --tiling-density 1 --probe-count 5000 --output-dir test_res --verbose --keep test/GenomeInfo.tab 2>test.log
```

## Evaluation signals

- TiledBaits.fna exists and contains at most the requested probe count (e.g., 13,500 with -n 13500); the tutorial run should reproduce the 5,000-probe test output.
- Balance report shows per-taxon probe counts distributed under the minimum tiling-density target with balanced numbers of baits per input genome.
- No blacklist-cross-reactive candidates remain after Filtering: no hits at >=75% identity over >=30 bp with e-value <0.01 against the supplied BLAST databases.
- Sepsis benchmark reproduction: 26,870 probes, 2.09% target coverage, 3.64x average depth, efficiency 29.3 for 1,926 PATRIC genomes (81 species, 35 genera).
- Intermediate files (CandidateBaitRegions.tab, FilteredCandidates.tab, RepSeq.fna, Assignment.tsv) are present and format-preserved when pipeline phases are run separately.

## Limitations

- HUBDesign requires annotated genomes; probes cannot be designed for unknown or unannotatable genomes.
- Enrichment is target-sequence dependent and varies for each probe set; no general guarantee of enrichment level.
- SA_BOND memory scales linearly with genome size and organism number, but time grows much more quickly with diversity (e.g., 161 genera of gut bacteria run far longer than 56 coronaviruses).
- Setting --tiling-density too high can make per-genome balancing impossible, yielding unexpectedly small probe sets.
- Earlier versions had a cluster-partitioning flaw (nodes split by nominal taxonomy rather than observed sequence similarity) affecting the published sepsis set; newer versions construct a cluster-assignment tree from observed clusters.
- Validation used artificial pooled samples, not patient or environmental samples; no contaminant blacklist databases were provided (suggested future capability).

## Evidence
