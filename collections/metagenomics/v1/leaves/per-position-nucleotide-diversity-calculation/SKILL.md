---
name: per-position-nucleotide-diversity-calculation
description: Use when you have a sorted .bam of reads mapped to a reference .fasta
  and need per-position nucleotide diversity plus scaffold-level coverage, breadth,
  and expected-breadth metrics for microdiversity profiling.
when_to_use_negative:
- Input is raw reads or an unsorted .sam/.bam rather than a sorted .bam aligned to
  a fixed reference .fasta — coverage/diversity profiling needs position-resolved
  alignments
- The goal is genome dereplication or ANI comparison between assemblies (e.g. dRep/skani
  territory), not within-genome microdiversity profiling
- Coverage is too low for positions to pass the min_cov threshold of 5 reads — diversity
  estimates will be undefined or unreliable at low coverage
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: Python package that performs the per-position coverage/diversity profiling,
    SNV calling, and gene localization; provides the expectedBreadth model and scaffold_info
    output
  repo: github.com/MrOlm/instrain
- name: Samtools
  role: Used by inStrain to interact with .bam/.sam files and calculate the coverage
    of each position in the provided nucleotide sequence file
- name: Numpy
  role: Vectorized computation of per-position clonality (sum of squared base frequencies)
    and nucleotide diversity over covered positions
- name: Pandas
  role: Assembles per-scaffold metrics into the scaffold_info.tsv table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:f698f20caffc7041c8559400b4a294c17f930e6aceecaf1df0d41e10ecb5f480
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Numpy
  - Pandas
  - Samtools
  - inStrain
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# per-position-nucleotide-diversity-calculation

## Summary

Compute per-position nucleotide diversity (1 − clonality, the sum of squared A/C/G/T base frequencies) across a reference genome from properly paired, filtered .bam alignments, then aggregate per-scaffold coverage, breadth, and diversity statistics as inStrain does for profiling co-occurring genome populations from metagenomes.

## When to use

Use when you have a sorted .bam of reads mapped to a reference .fasta and need per-position nucleotide diversity plus scaffold-level coverage, breadth, and expected-breadth metrics for microdiversity profiling.

## When NOT to use

- Input is raw reads or an unsorted .sam/.bam rather than a sorted .bam aligned to a fixed reference .fasta — coverage/diversity profiling needs position-resolved alignments
- The goal is genome dereplication or ANI comparison between assemblies (e.g. dRep/skani territory), not within-genome microdiversity profiling
- Coverage is too low for positions to pass the min_cov threshold of 5 reads — diversity estimates will be undefined or unreliable at low coverage

## Inputs

- Sorted .bam file of reads mapped to a reference genome (e.g. from Bowtie 2 or Samtools)
- Reference genome .fasta file matching the .bam

## Outputs

- scaffold_info.tsv with columns: scaffold name, length, coverage, median coverage, breadth, unmasked breadth, expected breadth, and nucleotide diversity
- Per-position base counts and nucleotide diversity values underlying gene-level and genome-wide microdiversity reporting

## How to apply

Load the sorted .bam with Samtools together with the reference .fasta and, for each scaffold, tally per-position read depth and A/C/G/T base counts from properly paired primary alignments; inStrain deliberately filters reads in the .bam to reduce erroneous mapping (removing un-paired reads by default, requiring proper orientation within expected insert size, a minimum mapQ, and a minimum nucleotide identity for pairs). Using Numpy, compute per-position nucleotide diversity as 1 − clonality (the sum of squared A/C/G/T base frequencies) at positions whose coverage exceeds the min_cov threshold of 5 reads — the same minimum read count inStrain applies to SNV calling. Aggregate per-scaffold statistics into Pandas: average and median coverage, breadth (fraction of reference bases covered by ≥1 read), unmasked breadth excluding masked reference bases, and nucleotide diversity. Compute expected breadth from the scaffold's mean coverage using the inStrain model expectedBreadth = -1*e^(-0.883*coverage)+1. Write the metrics as a tab-separated scaffold_info.tsv listing scaffold name, length, coverage, median coverage, breadth, unmasked breadth, expected breadth, and nucleotide diversity, which also feeds inStrain's later gene-level and genome-wide microdiversity reporting.

## Related tools

- **inStrain** (Python package that performs the per-position coverage/diversity profiling, SNV calling, and gene localization; provides the expectedBreadth model and scaffold_info output) — github.com/MrOlm/instrain
- **Samtools** (Used by inStrain to interact with .bam/.sam files and calculate the coverage of each position in the provided nucleotide sequence file)
- **Numpy** (Vectorized computation of per-position clonality (sum of squared base frequencies) and nucleotide diversity over covered positions)
- **Pandas** (Assembles per-scaffold metrics into the scaffold_info.tsv table)

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1
```

## Evaluation signals

- scaffold_info.tsv contains all metric columns: scaffold name, length, coverage, median coverage, breadth, unmasked breadth, expected breadth, and nucleotide diversity
- At 20x simulated mapping (e.g. pIRS-simulated reads), observed breadth ≈ 1, matching the documented expectation of 100% genome breadth (every base covered by ≥1 read) at 20x coverage
- Expected breadth from expectedBreadth = -1*e^(-0.883*coverage)+1 is close to observed breadth for uniform-coverage mappings
- Nucleotide diversity is only computed at positions with coverage greater than the min_cov threshold of 5 reads; positions below this cutoff carry no diversity value
- Diversity values fall within [0, 0.75] since nucleotide diversity is 1 minus the sum of squared A/C/G/T frequencies

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain, biasing per-position diversity near scaffold termini
- Coverage needed to detect minor alleles with 95% probability is nontrivial — low-coverage positions fail the default minimum of 5 reads and variant reads must exceed a null model (FDR default 1e-6)
- Careful filtering of reads in the .bam is required to reduce the probability of reads being erroneously mapped; multi-mapped reads from highly similar strains inflate apparent diversity unless genomes are de-replicated (98% ANI reduces multi-mapped reads)

## Evidence

- [readme] README quick start invocation: [evidence span withheld — anchor status: external_doc]
