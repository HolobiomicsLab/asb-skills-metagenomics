---
name: mock-community-strain-count-comparison
description: Use when benchmarking OTU-count fidelity of 16S pipelines on mock communities
  with known strain composition, to judge which pipeline's OTU count deviates least
  from truth.
when_to_use_negative:
- Input is already a feature/OTU table from real (non-mock) communities with no known
  ground-truth strain count — no fidelity denominator exists
- The analysis goal is taxonomic annotation accuracy rather than OTU-count fidelity
  against true strain numbers
- Mock reads are not highly overlapping 2 × 250 bp pairs; the benchmark was designed
  for datasets with highly overlapping paired-end reads
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: CDSnake
  role: Pipeline under test; launches CD-HIT-OTU-Miseq utilities to produce OTU tables
    (OTU.txt) from paired-end reads without merging
  repo: https://github.com/ydkondratenko/cdsnake
- name: CD-HIT-OTU-Miseq
  role: Underlying clustering approach evaluated; separate clustering of R1/R2 reads
    avoiding the reads merging process
- name: DADA2
  role: Baseline pipeline (via QIIME2) whose OTU counts on Balanced/HMP/Extreme are
    compared against true strain counts
- name: deblur
  role: Baseline pipeline (via QIIME2) whose OTU counts on Balanced/HMP/Extreme are
    compared against true strain counts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:e242938eded87b9e51cca3bf39b7524e25da1712370b2a9df05eb1ceb9fdf87f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ydkondratenko/cdsnake
  tools:
  - CD-HIT-OTU-Miseq
  - CDSnake
  - DADA2
  - deblur
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# mock-community-strain-count-comparison

## Summary

Benchmarks 16S OTU-retrieval pipelines (CDSnake/CD-HIT-OTU-Miseq, DADA2, deblur) on mock communities by computing |OTUs outputted − true strain count| per dataset (Balanced: 59, HMP: 21, Extreme: 27 strains) and comparing totals, reproducing Table 1 fidelity claims.

## When to use

Use when benchmarking OTU-count fidelity of 16S pipelines on mock communities with known strain composition, to judge which pipeline's OTU count deviates least from truth.

## When NOT to use

- Input is already a feature/OTU table from real (non-mock) communities with no known ground-truth strain count — no fidelity denominator exists
- The analysis goal is taxonomic annotation accuracy rather than OTU-count fidelity against true strain numbers
- Mock reads are not highly overlapping 2 × 250 bp pairs; the benchmark was designed for datasets with highly overlapping paired-end reads

## Inputs

- Reproduced Table 1 OTU counts per pipeline (CDSnake, DADA2, deblur) per dataset (Balanced, HMP, Extreme, All)
- True mock-community strain counts: Balanced = 59, HMP = 21, Extreme = 27
- Paired-end 2 × 250 bp FASTQ reads from the three mock datasets (for re-running pipelines)
- Formatted 16S reference database (Greengenes or SILVA, reformatted per CD-HIT-OTU-Miseq installation step 4)

## Outputs

- otu_count_deviations.csv with per-dataset absolute deviations |OTUs − true strains| and verdict columns closer_on_balanced_hmp, closer_on_extreme, fewer_otus_overall
- Per-dataset pipeline ranking by deviation (e.g. CDSnake first on Balanced and HMP; DADA2 first on Extreme)
- Overall OTU totals per pipeline across datasets

## How to apply

Load the reproduced Table 1 OTU counts (CDSnake, DADA2, deblur on Balanced, HMP, Extreme) plus the true mock strain counts — 59 for Balanced (57 bacteria and archaea nominal composition per the text), 21 for HMP, and 27 for Extreme — into a pandas DataFrame. For each pipeline–dataset pair compute the absolute deviation, e.g. Balanced: |91 − 59| = 32 for DADA2, |34 − 59| = 25 for deblur, |63 − 59| = 4 for CDSnake. Rank pipelines by deviation per dataset and compute overall totals across the three datasets (plus the 'All' row) to test the claim that CDSnake outputs fewer OTUs. Write the deviation table with per-claim verdict columns (closer_on_balanced_hmp, closer_on_extreme, fewer_otus_overall) to otu_count_deviations.csv. The rationale is that mock communities have a known ground-truth strain number, so OTU counts near the truth indicate the pipeline neither over- nor under-splits; CDSnake avoids read merging by separate clustering of paired reads, which the paper credits with discarding a smaller portion of reads.

## Related tools

- **CDSnake** (Pipeline under test; launches CD-HIT-OTU-Miseq utilities to produce OTU tables (OTU.txt) from paired-end reads without merging) — https://github.com/ydkondratenko/cdsnake
- **CD-HIT-OTU-Miseq** (Underlying clustering approach evaluated; separate clustering of R1/R2 reads avoiding the reads merging process)
- **DADA2** (Baseline pipeline (via QIIME2) whose OTU counts on Balanced/HMP/Extreme are compared against true strain counts)
- **deblur** (Baseline pipeline (via QIIME2) whose OTU counts on Balanced/HMP/Extreme are compared against true strain counts)

## Examples

```
cd input_folder && snakemake; python compare_otu_counts.py --table pooled/OTU.txt --true-strains Balanced=59,HMP=21,Extreme=27 --out otu_count_deviations.csv
```

## Evaluation signals

- Computed deviations must confirm CDSnake has the smallest |OTUs − true| on Balanced (4 vs DADA2=32, deblur=25) and HMP (38 vs DADA2=47, deblur=32)
- On Extreme, DADA2 (deviation 3) or deblur (9) must rank closer to the 27 true strains than CDSnake (15)
- The 'fewer OTUs overall' verdict must be False: CDSnake exceeds deblur on Balanced (55 vs 34), HMP (59 vs 53), and All (123 vs 39)
- Sanity check raw counts against Table 1 anchors: DADA2=91 and deblur=34 OTUs on Balanced; HMP timing/values consistent with the time-benchmark rows
- CSV schema check: each row carries dataset, pipeline, OTU count, true strain count, absolute deviation, and the three verdict columns

## Limitations

- The claim that CDSnake outputs fewer OTUs than both baselines only holds versus DADA2; CDSnake outputs more OTUs than deblur on Balanced, HMP, and All
- Ground truth is limited to the three mock datasets (Balanced, HMP, Extreme) with highly overlapping 2 × 250 bp reads; results may not generalize to other read lengths or real communities
- Software versions for DADA2, deblur, CD-HIT-OTU-MiSeq, and Snakemake are not stated in the paper, limiting reproducibility
- No accession or availability details are given for the three mock community datasets

## Evidence

- [abstract] on Balanced and HMP datasets number of OTUs outputted by CDSnake was closer to real number of strains which were used for mock community generation: "on Balanced and HMP datasets number of OTUs outputted by CDSnake was closer to real number of strains which were used for mock community generation, than those outputted by DADA2 and deblur."
- [results] Balanced community composition: "The Balanced community contained 57 bacteria and archaea at nominally equal frequencies"
- [results] Extreme community composition: "the Extreme community contained 27 bacterial strains at frequencies spanning five orders of magnitude"
- [results] DADA2 Balanced OTU count: [evidence span withheld — anchor status: not_found]
- [methods] 97% clustering threshold: "Remaining reads and fragments from annotated database are then clustered at 97% similarity to yields annotated clusters commonly named Operational Taxonomic Units, or OTU."
- [readme] Reads are unzipped fastq with R1/R2 naming; run via snakemake; output OTU table location: "Reads should be unzipped prior to pipeline usage (should have an extension “.fastq”). Forward and reverse reads should have “_R1_” and “_R2_” in filenames."
