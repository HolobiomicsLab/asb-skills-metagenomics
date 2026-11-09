---
name: read-pair-overlap-scoring
description: 'Use when a metagenomic assembly needs its initial divide step: you have
  interleaved Illumina FASTQ reads from a complex community and must partition them
  into small clusters that each correspond roughly to one species before local/global
  assembly.'
when_to_use_negative:
- Reads are long noisy PacBio/ONT data — StrainXpress targets next-generation (Illumina-type)
  short reads and the paper notes third-generation error rates remain unresolved
- A reference genome is available and reference-guided assembly is acceptable — this
  skill is a de novo, reference-free divide step
- You only need final contig sequences, not the intermediate read clustering — use
  the full StrainXpress pipeline including Local and Global Assembly instead
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: StrainXpress
  role: Implements the overlap-score sorting and single-linkage read clustering (the
    'Clustering Reads' stage, also reusable as 'XC' pre-processing)
  repo: https://github.com/HaploKit/StrainXpress
- name: minimap2
  role: 'Overlap stage: computes pairwise read overlaps and overlap scores from the
    interleaved FASTQ input'
  repo: https://github.com/lh3/minimap2
- name: HaploConduct
  role: Dependency of StrainXpress providing overlap-graph assembly components (SAVAGE/POLYTE
    lineage) used downstream
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:c753379b5f35d628ef295913ca605e7b3d5db7b59d785db7d2fc27da6dfc534a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - HaploConduct
  - StrainXpress
  - minimap2
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# read-pair-overlap-scoring

## Summary

Scores pairwise overlaps between metagenomic short reads with Minimap2 and merges the read pairs into species-coherent clusters via descending-score single-linkage clustering, forming the 'Clustering Reads' stage of StrainXpress's OLC divide-and-conquer pipeline.

## When to use

Use when a metagenomic assembly needs its initial divide step: you have interleaved Illumina FASTQ reads from a complex community and must partition them into small clusters that each correspond roughly to one species before local/global assembly.

## When NOT to use

- Reads are long noisy PacBio/ONT data — StrainXpress targets next-generation (Illumina-type) short reads and the paper notes third-generation error rates remain unresolved
- A reference genome is available and reference-guided assembly is acceptable — this skill is a de novo, reference-free divide step
- You only need final contig sequences, not the intermediate read clustering — use the full StrainXpress pipeline including Local and Global Assembly instead

## Inputs

- Interleaved paired-end Illumina FASTQ file (all_reads.fq) with records like @S0R0/1 and @S0R0/2
- Minimap2 overlap file with read-pair overlap scores (produced by the StrainXpress overlap stage)

## Outputs

- cluster_assignments.tsv: well-formed TSV mapping each read ID to exactly one non-empty cluster ID, with reads from the same species in the same cluster

## How to apply

Read the interleaved FASTQ (all_reads.fq) and run StrainXpress's overlap stage, which uses Minimap2 to produce a read-pair overlap-score file. Sort all overlap-score entries in descending order so the most compatible read pairs are at the top, then traverse the sorted list once, merging the clusters of two reads whenever a pair links two distinct clusters (single-linkage: distance between clusters is the shortest member-to-member distance). This gives O(n(log n + n)) = O(n^2) overall complexity; for large data sets use the -fast cluster mode recommended in the README. Write the final assignments as a TSV mapping each read ID to its cluster ID such that reads from the same species land in the same cluster; the paper reports this clustering is 46x faster than the earlier approach it replaced and can also be reused ('XC') as a pre-processing step for other assemblers.

## Related tools

- **StrainXpress** (Implements the overlap-score sorting and single-linkage read clustering (the 'Clustering Reads' stage, also reusable as 'XC' pre-processing)) — https://github.com/HaploKit/StrainXpress
- **minimap2** (Overlap stage: computes pairwise read overlaps and overlap scores from the interleaved FASTQ input) — https://github.com/lh3/minimap2
- **HaploConduct** (Dependency of StrainXpress providing overlap-graph assembly components (SAVAGE/POLYTE lineage) used downstream) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python ../scripts/strainxpress.py -fq all_reads.fq -fast
```

## Evaluation signals

- cluster_assignments.tsv exists and is well-formed TSV with exactly one assignment row per input read
- Every cluster ID in the output is non-empty
- Reads known to come from the same species (e.g. the 20 strains of 10 species in the low-complexity synthetic set) fall into the same cluster
- The number of clusters is species-scale rather than read-scale (paper reports cluster counts in the hundreds, e.g. 899 clusters, not thousands per strain)
- On a full run, the downstream pipeline completes, since 'Clustering Reads' output feeds the 'Local Assembly' and 'Global Assembly' stages

## Limitations

- Single-linkage merging with a full sort of all overlap pairs is O(n^2) overall, so very large data sets require the -fast cluster mode
- Short Illumina reads constrain downstream contig length, preventing strain-specific contigs that span whole genomes
- Performance depends on overlap-score quality from Minimap2; pairs with spuriously low scores are only merged late and may be missed within the single pass

## Evidence

- [readme] OLC paradigm framing: "StrainXpress is a de novo assembly method which base on overlap-layout-consensus (OLC) paradigm"
- [readme] Input format requirement: "The input file must be interleaved FASTQ"
- [results] Clustering speedup: "‘StrainXpress - Clustering’ is 46 times faster than the earlier work (22)"
- [results] Clustering as reusable pre-processing: "We considered the application of StrainXpress’s clustering procedure (referred to as ‘XC’ in Table 1) as a pre-processing step"
- [intro] Rationale for overlap-graph pipelines: "overcoming the computational bottlenecks of OG-based approaches is key to success when aiming at considerably improved haplotype/strain aware assemblies of metagenomes"
