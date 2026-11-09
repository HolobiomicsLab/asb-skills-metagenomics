---
name: discordant-read-pair-junction-selection
description: Use when paired-end shotgun NGS reads have been uniquely mapped to metagenomic
  reference genomes with BWA and junction pairs spanning two different references
  must be grouped into candidate HGT breakpoints.
when_to_use_negative:
- You already need exact breakpoint nucleotide positions — that requires the subsequent
  split-read refinement stage (get_acc_bkp), not this candidate-generation step.
- Input reads are not paired-end shotgun reads or the BAM contains multi-mapped/non-unique
  alignments; LEMON's junction-pair selection assumes unique-mapping BAM input.
- Breakpoints are already available (e.g. from LUMPY output) and only downstream strain
  reconstruction is needed.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: BWA
  role: Aligns paired-end shotgun reads against the metagenomic references to produce
    the unique-mapping BAM from which junction pairs are selected
  repo: https://github.com/lichen2018/LEMON
- name: DBSCAN
  role: Density-based clustering (Euclidean distance) of breakpoint-pair candidate
    points into candidate HGT breakpoints
- name: LEMON
  role: Implements the raw-breakpoint detection script (get_raw_bkp.py) that performs
    junction selection and DBSCAN clustering
  repo: https://github.com/lichen2018/LEMON
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:814ac1e1ad82406637b64e351dc4620a6d91af282049f7c764d074e26842af11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - DBSCAN
  - LEMON
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# discordant-read-pair-junction-selection

## Summary

In LEMON's raw-breakpoint stage, discordant read pairs whose two ends map to two different references are converted to 2D points and clustered with DBSCAN (Euclidean distance, eps = average insert size, minsample = 1) to generate candidate HGT breakpoints before split-read refinement.

## When to use

Use when paired-end shotgun NGS reads have been uniquely mapped to metagenomic reference genomes with BWA and junction pairs spanning two different references must be grouped into candidate HGT breakpoints.

## When NOT to use

- You already need exact breakpoint nucleotide positions — that requires the subsequent split-read refinement stage (get_acc_bkp), not this candidate-generation step.
- Input reads are not paired-end shotgun reads or the BAM contains multi-mapped/non-unique alignments; LEMON's junction-pair selection assumes unique-mapping BAM input.
- Breakpoints are already available (e.g. from LUMPY output) and only downstream strain reconstruction is needed.

## Inputs

- Unique-mapping BAM file from BWA alignment of paired-end shotgun metagenomic reads
- Metagenomic reference set (e.g. the paper's reference set S: 16,093 species, 1,246,881 scaffolds)
- Average insert size of the sequencing library (used as the DBSCAN eps parameter)

## Outputs

- raw_breakpoints.txt containing candidate HGT breakpoint pairs, one per retained DBSCAN cluster
- Cluster assignments of junction-pair candidate points (each cluster supported by at least one junction pair)

## How to apply

First align paired-end shotgun reads against the metagenomic reference set with BWA and keep the unique-mapping BAM (via `bwa mem -M`). From the mapped reads, select junction pairs whose two sides map to two different references, and transform each pair's two mapped positions into a point on a two-dimensional plane. Cluster these breakpoint-pair candidate points with the density-based DBSCAN algorithm using Euclidean distance, setting eps to the average insert size and minsample to 1. Retain every cluster supported by at least one junction pair and write the candidate breakpoint pairs to raw_breakpoints.txt; these raw candidates are then passed to LEMON's split-read stage to determine exact breakpoint positions. The rationale is that true HGT events yield a spatially dense cluster of discordant pairs at the breakpoint, while mapping noise appears as isolated points.

## Related tools

- **BWA** (Aligns paired-end shotgun reads against the metagenomic references to produce the unique-mapping BAM from which junction pairs are selected) — https://github.com/lichen2018/LEMON
- **DBSCAN** (Density-based clustering (Euclidean distance) of breakpoint-pair candidate points into candidate HGT breakpoints)
- **LEMON** (Implements the raw-breakpoint detection script (get_raw_bkp.py) that performs junction selection and DBSCAN clustering) — https://github.com/lichen2018/LEMON

## Examples

```
python LEMON/Scripts/get_raw_bkp.py -r metagenomic_reference.fa -u unique_reads.bam -o raw_breakpoints.txt -t 4
```

## Evaluation signals

- raw_breakpoints.txt is produced and contains candidate breakpoint pairs, each traceable to a DBSCAN cluster with at least one supporting junction pair
- DBSCAN parameters match the paper: eps equals the average insert size and minsample is 1
- Every clustered point corresponds to a junction pair whose two ends map to two different references in the unique-mapping BAM
- Candidate breakpoints are consumable by the next stage: the -b/--raw breakpoints input of LEMON/getAccBkp/get_acc_bkp for split-read refinement
- Spot-check that reported breakpoints lie at junctions between two distinct reference scaffolds/species rather than within a single reference

## Limitations

- With minsample = 1, even a single discordant pair forms a retained cluster, so false-positive candidates are expected and must be filtered by the downstream split-read accuracy stage.
- Only unique-mapping reads are usable; reads mapping ambiguously across the 1,246,881 scaffolds of the reference set are excluded from junction-pair evidence.
- Candidate breakpoints are approximate; the method depends on the subsequent split-read step (Smith-Waterman based) to determine exact positions.

## Evidence

- [methods] Firstly, we map paired-end shotgun reads against reference genomes using Burrows Wheeler Aligner (BWA).: "Firstly, we map paired-end shotgun reads against reference genomes using Burrows Wheeler Aligner (BWA)."
- [methods] Then, we select junction reads, whose two sides are mapped to two different references, from the set of mapped reads.: "Then, we select junction reads, whose two sides are mapped to two different references, from the set of mapped reads."
- [methods] We then apply the clustering algorithm DBSCAN [23] using Euclidean distance to cluster the breakpoint pair candidates.: "We then apply the clustering algorithm DBSCAN [23] using Euclidean distance to cluster the breakpoint pair candidates."
- [readme] usage: python LEMON/Scripts/get_raw_bkp.py [options]: "usage: python LEMON/Scripts/get_raw_bkp.py [options]"
- [methods] we apply Density-Based Spatial Clustering of Applications with Noise (DBSCAN) to find candidate HGT breakpoints.: "we apply Density-Based Spatial Clustering of Applications with Noise (DBSCAN) to find candidate HGT breakpoints."
