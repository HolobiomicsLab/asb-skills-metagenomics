---
name: viral-metagenome-genome-binning
description: Use when you have predicted viral scaffolds (e.g., from VIBRANT or VirSorter)
  plus multi-sample metagenomic read coverage and need to cluster scaffolds into vMAGs
  while controlling contamination.
when_to_use_negative:
- Input is an entire (unfiltered) metagenome of mostly non-viral sequences — vRhyme
  is built to run on viral sequences/scaffolds and whole-metagenome performance has
  not been fully evaluated
- Goal is binning microbial (bacterial/archaeal) genomes — MetaBat2 performed better
  than vRhyme on a synthetic dataset of predominately non-viral sequences
- Few or single input coverage samples are available when robust coverage variance
  comparisons are required for accurate binning
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: vRhyme
  role: 'Primary binning tool: coverage effect size comparisons, supervised ML feature
    classification, weighted network construction, and protein redundancy bin refinement'
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Prodigal
  role: ORF prediction on input scaffolds ('-m -p meta') feeding the protein redundancy
    scoring
  repo: https://github.com/hyattpd/Prodigal
- name: Mmseqs2
  role: Homology-based protein redundancy scoring used to select the best binning
    iteration and flag contaminated bins
  repo: https://github.com/soedinglab/MMseqs2
- name: Bowtie2
  role: Read alignment (--no-unal --no-discordant) to compute per-sample scaffold
    coverage
- name: BWA
  role: Alternative aligner (bwa mem) for coverage computation
  repo: https://github.com/lh3/bwa
- name: Mash
  role: Optional scaffold dereplication pre-binning (--method longest/composite)
  repo: https://github.com/marbl/mash
- name: MetaBat2
  role: Competing bacterial-oriented binning tool used as redundancy/contamination
    benchmark on GOV2
- name: CoCoNet
  role: Competing virus-specific binning tool used as benchmark on GOV2
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:eb81159bf0a0ea5833d26ffb1d2dc7539a45afce7bc2fc833c4c23001b643137
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/AnantharamanLab/vRhyme
  repo_url: https://github.com/AnantharamanLab/vRhyme
  tools:
  - BWA
  - Bowtie2
  - CoCoNet
  - Mash
  - MetaBat2
  - Mmseqs2
  - Prodigal
  - Python
  - Samtools
  - vRhyme
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# viral-metagenome-genome-binning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Bins viral scaffolds into viral metagenome-assembled genomes (vMAGs) using vRhyme, which combines supervised machine learning (neural network + Extra Trees) on sequence features with single- or multi-sample read coverage effect size comparisons, refined via a protein redundancy scoring mechanism exploiting the observation that viruses seldom encode redundant genes.

## When to use

Use when you have predicted viral scaffolds (e.g., from VIBRANT or VirSorter) plus multi-sample metagenomic read coverage and need to cluster scaffolds into vMAGs while controlling contamination.

## When NOT to use

- Input is an entire (unfiltered) metagenome of mostly non-viral sequences — vRhyme is built to run on viral sequences/scaffolds and whole-metagenome performance has not been fully evaluated
- Goal is binning microbial (bacterial/archaeal) genomes — MetaBat2 performed better than vRhyme on a synthetic dataset of predominately non-viral sequences
- Few or single input coverage samples are available when robust coverage variance comparisons are required for accurate binning

## Inputs

- Viral scaffold nucleotide FASTA (virus predictions from VIBRANT/VirSorter, length-filtered, optionally dereplicated)
- Sorted BAM files of metagenomic reads aligned to scaffolds (Bowtie2 or BWA mem) for per-sample coverage
- Optional pre-computed Prodigal protein and gene files in 'name_#' format (-p and -g)
- Multi-sample metagenomic read sets for coverage variance comparisons

## Outputs

- vRhyme_best_bins.#.membership.tsv (scaffold-to-bin assignments)
- vRhyme_best_bins.#.summary.tsv (bin summaries)
- vRhyme_bin_scoring.tsv (per-iteration protein redundancy scores for best-iteration selection)
- vMAG FASTA files and per-bin redundant protein counts for contamination assessment
- CheckV completeness/contamination assessments of recovered vMAGs

## How to apply

Supply the length-filtered (≥5 kb for GOV2; ≥2000 bp for skin) viral scaffold FASTA as -i and sorted BAM coverage files as -b to vRhyme, optionally removing circular scaffolds (terminal repeats ≥20 bp with ≤2 mismatches) and dereplicating with Mash/Nucmer first. Run vRhyme with the hybrid machine learning model (neural network + Extra Trees), --bin_size 2, --iter 20, --cov 0.80, dynamic --max_edges 3-6, default coverage outlier removal (--outliers 4.0) and 150 bp end masking, letting Prodigal predict ORFs ('-m -p meta') and Mmseqs2 score protein redundancy across iterations. Select the best iteration from vRhyme_bin_scoring.tsv and use vRhyme_best_bins.#.membership.tsv and vRhyme_best_bins.#.summary.tsv to count binned scaffolds and bins. Classify bins with 0-1 redundant proteins as low-contamination vMAGs (filter >6 as likely contaminated, allowing ~10 for NCLDVs), and assess completeness/contamination with CheckV. Benchmark or validate by comparing scaffold counts, bin counts, redundant-protein totals, and CheckV distributions against published values (e.g., GOV2: 56,642 scaffolds into 13,175 bins, 15,106 redundant proteins vs 29,334 for MetaBat2 and 71,364 for CoCoNet; human skin: 10,601 scaffolds into 849 vMAGs, 762 low-redundancy).

## Related tools

- **vRhyme** (Primary binning tool: coverage effect size comparisons, supervised ML feature classification, weighted network construction, and protein redundancy bin refinement) — https://github.com/AnantharamanLab/vRhyme
- **Prodigal** (ORF prediction on input scaffolds ('-m -p meta') feeding the protein redundancy scoring) — https://github.com/hyattpd/Prodigal
- **Mmseqs2** (Homology-based protein redundancy scoring used to select the best binning iteration and flag contaminated bins) — https://github.com/soedinglab/MMseqs2
- **Bowtie2** (Read alignment (--no-unal --no-discordant) to compute per-sample scaffold coverage)
- **BWA** (Alternative aligner (bwa mem) for coverage computation) — https://github.com/lh3/bwa
- **Mash** (Optional scaffold dereplication pre-binning (--method longest/composite)) — https://github.com/marbl/mash
- **MetaBat2** (Competing bacterial-oriented binning tool used as redundancy/contamination benchmark on GOV2)
- **CoCoNet** (Competing virus-specific binning tool used as benchmark on GOV2)

## Examples

```
vRhyme -i viral_scaffolds.fna -b sample*.sorted.bam -o out/ --bin_size 2 --iter 20 --model hybrid --cov 0.80 --outliers 4.0
```

## Evaluation signals

- vRhyme_bin_scoring.tsv exists and identifies a best iteration; membership and summary TSVs present in the output hierarchy
- On GOV2, reproduced statistics match Supplementary Table S7: 56,642 scaffolds into 13,175 bins, 15,106 redundant proteins, 11,909 low-contamination bins
- On human skin, run yields 849 vMAGs total with 762 having 0-1 redundant proteins, covering 2,794 of 10,601 scaffolds
- Redundant-protein totals compare favorably to MetaBat2 (29,334) and CoCoNet (71,364) on the same GOV2 scaffolds
- CheckV confirms expected completeness, e.g., the 22-scaffold Herelleviridae vMAG reaches ~100% completeness

## Limitations

- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome
- Binning non-viral genomes is an anticipated limitation; MetaBat2 outperformed vRhyme on a predominantly non-viral synthetic dataset
- Performance depends on the number of input coverage samples; vRhyme performs optimally with multiple samples for robust coverage variance comparisons
- A small number of redundant proteins may be acceptable in special cases — bins with up to ~10 redundant proteins can still be valid for NCLDVs

## Evidence

- [abstract] ml features: [evidence span withheld — anchor status: not_found]
- [readme] update notes: [evidence span withheld — anchor status: not_found]
