---
name: dbscan-breakpoint-cluster-analysis
description: Use when paired-end shotgun NGS reads mapped with BWA yield junction
  read pairs whose two sides map to two different references, and candidate HGT breakpoint
  pairs must be generated before split-read refinement.
when_to_use_negative:
- Accurate breakpoint positions are already needed — this skill only generates raw
  candidates; use LEMON's split-read refinement stage (get_acc_bkp) afterwards instead.
- Reads are not uniquely mapped or the alignment is not from paired-end shotgun data,
  so no two-reference junction pairs can be extracted.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: LEMON
  role: Pipeline implementing the raw HGT breakpoint detection stage via get_raw_bkp.py
  repo: https://github.com/lichen2018/LEMON
- name: DBSCAN
  role: Density-based clustering of breakpoint-pair candidate points with Euclidean
    distance (eps = average insert size, minsample = 1)
- name: BWA
  role: Aligns paired-end shotgun reads against reference genomes to produce the unique-mapping
    BAM from which junction pairs are selected
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:6a630939294cc7d119ab57a5acf0b794a74ce422782326ed30a5164955c19062
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - DBSCAN
  - LEMON
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# dbscan-breakpoint-cluster-analysis

## Summary

Cluster discordant read-pair evidence into candidate HGT breakpoints using DBSCAN with Euclidean distance, as implemented in LEMON's raw-breakpoint detection stage on shotgun metagenomic alignments.

## When to use

Use when paired-end shotgun NGS reads mapped with BWA yield junction read pairs whose two sides map to two different references, and candidate HGT breakpoint pairs must be generated before split-read refinement.

## When NOT to use

- Accurate breakpoint positions are already needed — this skill only generates raw candidates; use LEMON's split-read refinement stage (get_acc_bkp) afterwards instead.
- Reads are not uniquely mapped or the alignment is not from paired-end shotgun data, so no two-reference junction pairs can be extracted.

## Inputs

- unique-mapping BAM file of paired-end shotgun reads aligned with BWA against a metagenomic reference
- metagenomic reference (multi-genome FASTA, e.g. 16,093 species / 1,246,881 scaffolds from NCBI bacterial genomes)

## Outputs

- raw_breakpoints.txt: clustered candidate HGT breakpoint pairs, each cluster supported by at least one junction read pair

## How to apply

Load the unique-mapping BAM produced by aligning paired-end shotgun reads against the metagenomic reference with BWA (bwa mem). Select junction reads whose two sides map to two different references, and transform each pair's two mapped positions into a point on a two-dimensional plane. Apply the DBSCAN density-based clustering algorithm (via scikit-learn) with Euclidean distance, setting eps to the average insert size and minsample to 1. Retain clusters supported by at least one junction pair and write their candidate breakpoint pairs to raw_breakpoints.txt; this raw file is then consumed by LEMON's accurate-breakpoint stage (get_acc_bkp), which refines positions with split reads.

## Related tools

- **LEMON** (Pipeline implementing the raw HGT breakpoint detection stage via get_raw_bkp.py) — https://github.com/lichen2018/LEMON
- **DBSCAN** (Density-based clustering of breakpoint-pair candidate points with Euclidean distance (eps = average insert size, minsample = 1))
- **BWA** (Aligns paired-end shotgun reads against reference genomes to produce the unique-mapping BAM from which junction pairs are selected)

## Examples

```
python LEMON/Scripts/get_raw_bkp.py -r metagenomic_reference.fasta -u unique_reads.bam -o raw_breakpoints.txt -t 4
```

## Evaluation signals

- Every reported breakpoint cluster is supported by at least one junction pair (minsample = 1 criterion).
- Output raw_breakpoints.txt is produced by get_raw_bkp.py and is accepted as the -b argument of get_acc_bkp without format errors.
- Clustered points derive only from read pairs whose two sides map to two different references.
- Eps equals the average insert size of the sequencing library, so clusters roughly span insert-size-scale regions on the 2-D breakpoint plane.

## Limitations

- This stage yields only approximate (raw) breakpoints; exact positions require the subsequent split-read (Smith-Waterman/ssw-py based) refinement step.
- Clusters depend on the quality and uniqueness of the BWA alignment and on the insert-size estimate used for eps; mis-set eps can merge or split true breakpoint candidates.

## Evidence

- [methods] DBSCAN clusters junction-pair points with Euclidean distance: "We then apply the clustering algorithm DBSCAN [23] using Euclidean distance to cluster the breakpoint pair candidates."
- [methods] DBSCAN finds candidate HGT breakpoints: "we apply Density-Based Spatial Clustering of Applications with Noise (DBSCAN) to find candidate HGT breakpoints."
- [methods] Junction reads with sides on different references are selected: "Then, we select junction reads, whose two sides are mapped to two different references, from the set of mapped reads."
- [methods] Split reads refine exact breakpoint positions afterwards: "Fourthly, we utilize split reads to get the exact positions of HGT breakpoints."
- [readme] README documents get_raw_bkp.py as the raw-breakpoint detection entry point: "usage: python LEMON/Scripts/get_raw_bkp.py [options]"
