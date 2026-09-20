---
name: two-dimensional-coordinate-transform-of-mapped-read-pairs
description: Use when paired-end shotgun reads mapped with BWA yield junction pairs
  whose two sides map to different references, and the discordant pairs must be converted
  into 2D points for density-based clustering of HGT breakpoint candidates.
when_to_use_negative:
- Read pairs map concordantly to a single reference (no junction pairs across different
  references) — there are no discordant coordinates to transform
- Downstream analysis needs exact breakpoint positions from split reads — this transform
  only generates raw candidates and precedes LEMON's accurate breakpoint-detection
  step
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: LEMON
  role: Implements the junction-pair selection and 2D coordinate transform in the
    raw breakpoint-detection script get_raw_bkp.py
  repo: https://github.com/lichen2018/LEMON
- name: DBSCAN
  role: Clusters the transformed 2D breakpoint-pair candidate points using Euclidean
    distance with eps = average insert size and minsample = 1
- name: BWA
  role: Aligns paired-end shotgun reads against the metagenomic reference to produce
    the unique-mapping BAM whose discordant pairs feed the transform
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:2cf5b253fa4577fdbc8eb648c4e69200e1670428af5d0d18e1d44fca1926da0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - DBSCAN
  - LEMON
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Two-dimensional coordinate transform of mapped read pairs

## Summary

Transforms each discordant junction pair's two mapped positions (one on each of two different references) into a single point on a two-dimensional plane, so that HGT breakpoint-pair candidates can be clustered geometrically with DBSCAN in LEMON's raw breakpoint-detection stage.

## When to use

Use when paired-end shotgun reads mapped with BWA yield junction pairs whose two sides map to different references, and the discordant pairs must be converted into 2D points for density-based clustering of HGT breakpoint candidates.

## When NOT to use

- Read pairs map concordantly to a single reference (no junction pairs across different references) — there are no discordant coordinates to transform
- Downstream analysis needs exact breakpoint positions from split reads — this transform only generates raw candidates and precedes LEMON's accurate breakpoint-detection step

## Inputs

- BAM file of unique-mapping paired-end reads produced by BWA mem against the metagenomic reference
- Metagenomic reference (set of donor/receptor genomes)
- Junction pairs whose two sides map to two different references

## Outputs

- Two-dimensional coordinate representation of breakpoint-pair candidates (one point per junction pair)
- DBSCAN clusters of breakpoint-pair candidates (eps = average insert size, minsample = 1)
- raw_breakpoints.txt containing candidate HGT breakpoint pairs for downstream split-read refinement

## How to apply

After aligning paired-end shotgun reads against the metagenomic reference with BWA and extracting the unique-mapping BAM, select junction pairs whose two sides map to two different references. For each such pair, take the two mapped positions (one per reference) and encode them as a single point on a two-dimensional plane, so each junction pair becomes one coordinate in 2D space. Apply DBSCAN with Euclidean distance on these transformed points, setting eps to the average insert size and minsample to 1, and retain clusters supported by at least one junction pair; the surviving candidate breakpoint pairs are written to raw_breakpoints.txt. The rationale is that true HGT breakpoints accumulate many discordant read pairs mapping to nearby positions on both references, forming dense clusters, while noise points remain isolated and are discarded by the density-based criterion.

## Related tools

- **LEMON** (Implements the junction-pair selection and 2D coordinate transform in the raw breakpoint-detection script get_raw_bkp.py) — https://github.com/lichen2018/LEMON
- **DBSCAN** (Clusters the transformed 2D breakpoint-pair candidate points using Euclidean distance with eps = average insert size and minsample = 1)
- **BWA** (Aligns paired-end shotgun reads against the metagenomic reference to produce the unique-mapping BAM whose discordant pairs feed the transform)

## Examples

```
python LEMON/Scripts/get_raw_bkp.py -r metagenomic_reference.fa -u unique_reads.bam -o raw_breakpoints.txt
```

## Evaluation signals

- Each retained breakpoint cluster must be supported by at least one junction pair, per LEMON's candidate-generation criterion
- Transformed points should be one-per-junction-pair: the number of 2D points equals the number of junction pairs whose sides map to two different references
- DBSCAN parameters must match the paper's setting: eps equal to the average insert size and minsample set to 1
- Output must be written to raw_breakpoints.txt and be consumable as the -b raw breakpoints input of LEMON/getAccBkp/get_acc_bkp

## Limitations

- Raw breakpoints from this clustering are approximate; exact positions require the subsequent split-read-based accurate breakpoint-detection step
- With minsample = 1, any single junction pair forms its own cluster, so false candidates may pass into raw_breakpoints.txt and rely on later refinement
- The transform assumes paired-end shotgun data with a defined insert-size distribution; eps depends on the average insert size of the library

## Evidence

- [methods] transform each pair's two mapped positions into a point on a two-dimensional plane: [evidence span withheld — anchor status: not_found]
- [methods] DBSCAN with Euclidean distance to cluster the breakpoint pair candidates: "We then apply the clustering algorithm DBSCAN [23] using Euclidean distance to cluster the breakpoint pair candidates."
- [methods] select junction reads, whose two sides are mapped to two different references: "Then, we select junction reads, whose two sides are mapped to two different references, from the set of mapped reads."
- [methods] map paired-end shotgun reads against reference genomes using Burrows Wheeler Aligner (BWA): "Firstly, we map paired-end shotgun reads against reference genomes using Burrows Wheeler Aligner (BWA)."
- [readme] -b FILE raw breakpoints file (README): "-b        FILE  raw breakpoints file"
