---
name: read-alignment-against-multiple-reference-genomes
description: Use when reproducing or benchmarking the original alignment-based AFS
  pipeline on food metagenomics samples (e.g. KAL_D, Kal/KLyo on the AFS10 reference
  set), or when establishing the runtime/accuracy baseline that AFS-MetaCache is compared
  to.
when_to_use_negative:
- You only need classification/quantification speed at modern throughput — AFS-MetaCache/MetaCacheSpark
  queries at 17.1 MR/m versus 0.04 MR/m here, so alignment is the wrong choice for
  large collections
- Reference genome collection is large or growing — the per-genome alignment strategy
  scales linearly with the number of genomes and was the motivation for the alignment-free
  substitute
- Input is already a classified/quantified abundance table rather than raw reads needing
  alignment
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: BWA
  role: Read alignment tool applied separately to each of the 10 AFS10 reference genomes
    in the legacy AFS pipeline
- name: AFS-MetaCache
  role: Alignment-free minhashing substitute used as the speed/accuracy baseline for
    comparison in Tables 9-10
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Apache Spark implementation of the same minhashing approach for cluster-scale
    comparison
  repo: https://github.com/jmabuin/MetaCacheSpark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:7dfc0c93bfccb3f8d207d04b4ede90e46188fdb61f0cc9c98a36fa4a05775201
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - BWA
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# read-alignment-against-multiple-reference-genomes

## Summary

Reproduces the legacy alignment-based AFS pipeline, in which sequencing reads are aligned with BWA against each considered reference genome separately, to quantify ingredient composition and benchmark it against the k-mer based AFS-MetaCache approach (Tables 9-10).

## When to use

Use when reproducing or benchmarking the original alignment-based AFS pipeline on food metagenomics samples (e.g. KAL_D, Kal/KLyo on the AFS10 reference set), or when establishing the runtime/accuracy baseline that AFS-MetaCache is compared to.

## When NOT to use

- You only need classification/quantification speed at modern throughput — AFS-MetaCache/MetaCacheSpark queries at 17.1 MR/m versus 0.04 MR/m here, so alignment is the wrong choice for large collections
- Reference genome collection is large or growing — the per-genome alignment strategy scales linearly with the number of genomes and was the motivation for the alignment-free substitute
- Input is already a classified/quantified abundance table rather than raw reads needing alignment

## Inputs

- FASTQ/FASTA sequencing reads of the calibrator sausage dataset KAL_D and food samples Kal and KLyo
- The 10 reference genome FASTA files of the AFS10 database, each aligned separately
- Reference BWA index build per genome

## Outputs

- Per-genome/per-sample SAM/BAM alignments of reads against each of the 10 AFS10 reference genomes
- Table with database build time, query speed (MR/m), Σ FP and Σ Dev for the BWA-based pipeline vs. AFS-MetaCache, reproducing Tables 9-10 (saved as afs_pipeline_comparison_tables_9_10.tsv)

## How to apply

For each of the 10 reference genomes in the AFS10 database, run BWA separately against the reads of each sample (e.g. the KAL_D calibrator sausage dataset and the Kal/KLyo samples), following the previous alignment-based AFS pipeline; the paper notes this read alignment tool can be BWA, Bowtie2, or CUSHAW. Record the total index construction time and query speed in reads/minute (MR/m). From the resulting alignments, compute quantification accuracy metrics: the sum of false positives (Σ FP) and the sum of deviations (Σ Dev) per sample. Tabulate build time, query speed, Σ FP and Σ Dev side by side with AFS-MetaCache values (which build in 47 m and query at 17.1 MR/m) to reproduce Tables 9-10. Expect the alignment route to need ~7 h index construction and reach only ~0.04 MR/m on the AFS10 database, i.e. it is orders of magnitude slower than the minhashing-based substitute.

## Related tools

- **BWA** (Read alignment tool applied separately to each of the 10 AFS10 reference genomes in the legacy AFS pipeline)
- **AFS-MetaCache** (Alignment-free minhashing substitute used as the speed/accuracy baseline for comparison in Tables 9-10) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Apache Spark implementation of the same minhashing approach for cluster-scale comparison) — https://github.com/jmabuin/MetaCacheSpark

## Examples

```
bwa index genome_01.fa; bwa mem -t 16 genome_01.fa KAL_D_R1.fq KAL_D_R2.fq > KAL_D_vs_genome_01.sam  # repeat for each of the 10 AFS10 reference genomes
```

## Evaluation signals

- Alignments must be produced against each of the 10 AFS10 reference genomes individually, not a concatenated reference
- Reproduced index construction time ≈ 7 h and query speed ≈ 0.04 MR/m on the AFS10 database with KAL_D
- Σ FP and Σ Dev computed identically for both pipelines on the same samples so values are directly comparable
- Tabulated build time, query speed, Σ FP, and Σ Dev match the published Tables 9 and 10 (output_matches_reference)
- Speed-up ratio versus AFS-MetaCache (47 m build, 17.1 MR/m query) must be at least two orders of magnitude in query speed

## Limitations

- The alignment-based pipeline is orders-of-magnitude slower than AFS-MetaCache: 7 h index construction and 0.04 MR/m query speed versus 47 m and 17.1 MR/m
- Requires running a separate alignment for every considered reference genome (BWA, Bowtie2, or CUSHAW), which does not scale to growing genome collections
- No software version numbers for benchmarked tools are given in the article, complicating exact reproduction

## Evidence

- [intro] the AFS pipeline relies on applying a read alignment tool (such as BWA [7–9], Bowtie2 [10], or CUSHAW [11]) for each considered reference genome: "the AFS pipeline relies on applying a read alignment tool (such as BWA [7–9], Bowtie2 [10], or CUSHAW [11]) for each considered reference genome"
- [abstract] orders of magnitude faster than alignment pipeline: "It is orders-of-magnitude faster than our previous alignment-based AFS pipeline."
- [discussion] AFS-MetaCache and MetaCacheSpark are fastest in terms of database construction times: "AFS-MetaCache and MetaCacheSpark are fastest in terms of database construction times"
