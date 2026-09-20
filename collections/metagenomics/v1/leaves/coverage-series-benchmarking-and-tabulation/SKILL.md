---
name: coverage-series-benchmarking-and-tabulation
description: Use when you have a series of metagenome datasets that differ only in
  the sequencing coverage of known spike-in strains (e.g. Salmonella at 5x-40x) and
  need to quantify how coverage affects strain-aware assembly accuracy against a known
  ground truth.
when_to_use_negative:
- The input is a single metagenome sample without a ground-truth reference or spike-in
  design — accuracy against known references cannot be computed with metaQuast.
- The question is de novo strain discovery in a complex community (>1000 strains)
  rather than a controlled coverage-vs-accuracy benchmark against spiked-in references.
- Reads are third-generation (long, high-error) reads — the article notes the high
  error rate of third-generation sequencing reads introduces issues that remain unresolved.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: StrainXpress
  role: De novo strain-aware assembler run on each spike-in FASTQ dataset at every
    coverage level; outputs stageb/final_contigs.fasta
  repo: https://github.com/HaploKit/StrainXpress
- name: metaQuast
  role: Evaluates each of the eight contig sets against the known Salmonella spike-in
    references to extract genome fraction and misassembly-related metrics
- name: MEGAHIT
  role: Second-best comparison assembler at 5x coverage (48.26% genome reconstruction
    vs 70.26% for StrainXpress)
- name: minimap2
  role: Dependency of StrainXpress used within its OLC-based assembly pipeline
  repo: https://github.com/lh3/minimap2
- name: HaploConduct
  role: Dependency of StrainXpress installation
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:c2c42e5751c71c5a2c6ff432d85916e40b1d7baf3cd1a61a04a080cbfd5017c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - HaploConduct
  - MEGAHIT
  - StrainXpress
  - metaQuast
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# coverage-series-benchmarking-and-tabulation

## Summary

Benchmark a strain-aware metagenome assembler across a controlled coverage sweep by assembling spike-in datasets at incrementally increasing coverage (5x to 40x in steps of 5x) and tabulating metaQuast-derived accuracy metrics per coverage level into a single CSV.

## When to use

Use when you have a series of metagenome datasets that differ only in the sequencing coverage of known spike-in strains (e.g. Salmonella at 5x-40x) and need to quantify how coverage affects strain-aware assembly accuracy against a known ground truth.

## When NOT to use

- The input is a single metagenome sample without a ground-truth reference or spike-in design — accuracy against known references cannot be computed with metaQuast.
- The question is de novo strain discovery in a complex community (>1000 strains) rather than a controlled coverage-vs-accuracy benchmark against spiked-in references.
- Reads are third-generation (long, high-error) reads — the article notes the high error rate of third-generation sequencing reads introduces issues that remain unresolved.

## Inputs

- Eight strain-mixing spike-in datasets (real gut metagenome reads plus simulated Salmonella strain reads at 5x, 10x, 15x, 20x, 25x, 30x, 35x and 40x spike-in coverage)
- Interleaved paired-end FASTQ file per dataset (e.g. all_reads.fq)
- Known Salmonella spike-in reference genomes (ground truth) for metaQuast evaluation

## Outputs

- final_contigs.fasta per coverage level (from the stageb folder of each StrainXpress run)
- coverage_vs_accuracy.csv with one row per spike-in coverage level (5x-40x) and columns of metaQuast-derived accuracy metrics (genome fraction, misassembly-related quantities)

## How to apply

Obtain the eight strain-mixing spike-in datasets (real gut metagenome reads spiked with simulated Salmonella strain reads at 5x, 10x, 15x, 20x, 25x, 30x, 35x and 40x spike-in coverage) and format each as a single interleaved paired-end FASTQ file. Assemble each dataset with StrainXpress via 'python strainxpress.py -fq all_reads.fq', adding the '-fast' flag for the larger datasets as the README recommends for big data sets, and collect the 'stageb/final_contigs.fasta' from each run. Evaluate each of the eight contig sets against the known Salmonella spike-in references with metaQuast, extracting the same accuracy criteria (genome fraction, misassembly-related quantities) used in the paper's tables. Compile one CSV row per coverage level (5x-40x) so that the coverage-accuracy trend (e.g. StrainXpress reconstructs 70.26% of spiked-in Salmonella genomes at 5x versus 48.26% for second-best MEGAHIT, and Genome Fraction reaches 84.29% from 15x onwards) is directly tabulated and comparable.

## Related tools

- **StrainXpress** (De novo strain-aware assembler run on each spike-in FASTQ dataset at every coverage level; outputs stageb/final_contigs.fasta) — https://github.com/HaploKit/StrainXpress
- **metaQuast** (Evaluates each of the eight contig sets against the known Salmonella spike-in references to extract genome fraction and misassembly-related metrics)
- **MEGAHIT** (Second-best comparison assembler at 5x coverage (48.26% genome reconstruction vs 70.26% for StrainXpress))
- **minimap2** (Dependency of StrainXpress used within its OLC-based assembly pipeline) — https://github.com/lh3/minimap2
- **HaploConduct** (Dependency of StrainXpress installation) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python strainxpress.py -fq all_reads.fq -fast
```

## Evaluation signals

- coverage_vs_accuracy.csv exists and contains all eight coverage levels from 5x to 40x
- Each CSV row carries metaQuast accuracy metrics computed for the spiked-in Salmonella strains
- Each StrainXpress run produced a stageb/final_contigs.fasta file
- At 5x coverage, genome reconstruction should be around 70.26% for StrainXpress (vs 48.26% for MEGAHIT), and Genome Fraction should reach 84.29% from 15x coverage onwards
- Input FASTQ files are interleaved paired-end in the format expected by StrainXpress (@read/1 followed by @read/2)

## Limitations

- The coverage sweep is specific to Salmonella strains spiked into eight real gut metagenome datasets; transferability of the 5x-40x accuracy trend to other species or communities is not established.
- No specific version numbers for StrainXpress or competitor tools are stated, creating a reproducibility gap.
- Short Illumina-type reads constrain contig length, preventing computation of strain-specific contigs spanning whole genomes; the -fast flag is recommended for large datasets but may trade accuracy for speed.

## Evidence

- [results] the 8 data sets varied in terms of coverage of the spiked-in strains, ranging from 5× to 40× in steps of 5×: "the 8 data sets varied in terms of coverage of the spiked-in strains, ranging from 5× to 40× in steps of 5×"
- [results] metaQuast evaluation: "evaluated the quality of the assemblies of the 10 spiked in salmonella strains using metaQuast"
- [results] Quast criteria: "Quantities referring to the criteria listed in the corresponding Tables were determined using Quast (40)"
- [readme] fast mode for big datasets: "When a data set is big, we recommend to use the fast cluster method:"
- [readme] interleaved FASTQ input format: "The input file must be interleaved FASTQ and format like below:"
