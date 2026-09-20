---
name: 16s-rrna-abundance-threshold-filtering
description: Use when running HashSeq's inferTrueSequences on Illumina 16S rRNA amplicon
  FASTQ data and a filtering threshold must be set to remove low-abundance parent
  variants before LOESS-based error modeling and variant calling.
when_to_use_negative:
- The goal is detecting true low-abundance sequence variants — HashSeq is deliberately
  conservative and the paper recommends more sensitive algorithms such as DADA2 for
  low-abundance variants.
- Input is already a filtered count table (e.g., SvTable.txt from a prior inferTrueSequences
  run) rather than raw/trimmed FASTQ files.
- The analysis requires modeling indels or multi-mismatch variants; HashSeq only considers
  one-mismatch children and treats indel events as separate parent sequences.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: HashSeq
  role: R package whose inferTrueSequences function applies the abundance threshold
    filter to remove variants with fewer than the threshold total reads
  repo: https://github.com/FarnazFouladi/HashSeq
- name: R
  role: Runtime environment in which HashSeq is installed via devtools and inferTrueSequences/makeLoessPlot
    are executed
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:6355f97eb1e26a07202b90339ad89163af2b7997e047cb0e11159462af933227
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# 16S rRNA abundance threshold filtering

## Summary

Removes 16S rRNA gene sequence variants whose total reads across all samples fall below a parent-abundance threshold (default 1,000 reads; 10,000 for the largest data sets) during HashSeq variant inference, so that downstream inference and the LOESS background error model operate only on well-supported parents.

## When to use

Use when running HashSeq's inferTrueSequences on Illumina 16S rRNA amplicon FASTQ data and a filtering threshold must be set to remove low-abundance parent variants before LOESS-based error modeling and variant calling.

## When NOT to use

- The goal is detecting true low-abundance sequence variants — HashSeq is deliberately conservative and the paper recommends more sensitive algorithms such as DADA2 for low-abundance variants.
- Input is already a filtered count table (e.g., SvTable.txt from a prior inferTrueSequences run) rather than raw/trimmed FASTQ files.
- The analysis requires modeling indels or multi-mismatch variants; HashSeq only considers one-mismatch children and treats indel events as separate parent sequences.

## Inputs

- Input directory of paired, primer-trimmed, 250-base-trimmed 16S rRNA FASTQ files (e.g., the zymo mock community from PRJEB24409)
- Numeric abundance threshold (e.g., 1000; 10000 for very large data sets)
- childrenProperties.txt from HashSeq output (mean and SD of one-mismatch children per cluster with parent abundances) for choosing the threshold

## Outputs

- sequences.fasta containing parents above the threshold plus significant one-mismatch children
- SvTable.txt count table restricted to threshold-passing sequences for downstream taxonomic assignment
- OneMismatchCluster.txt, childrenProperties.txt, and InferenceTestSignificantChildren.txt from the same inferTrueSequences run

## How to apply

Set the threshold parameter in inferTrueSequences(inputDir, outputDir, threshold); HashSeq then removes sequence variants with total reads less than the threshold across all samples. Choose the threshold guided by the makeLoessPlot visualization of childrenProperties.txt, which plots the mean and standard deviation of one-mismatch children against parent abundance; the LOESS regression fits well above a parent abundance of 1,000 total reads, so 1,000 is the default for most data sets and 10,000 is used for the largest data set. The output directory receives sequences.fasta and SvTable.txt containing only parents above the threshold plus significant one-mismatch children. Rationale: parents above the threshold have a depth-dependent, normally distributed background error that the normal-plus-LOESS model captures, so variants below the cutoff cannot be reliably distinguished from sequencing error.

## Related tools

- **HashSeq** (R package whose inferTrueSequences function applies the abundance threshold filter to remove variants with fewer than the threshold total reads) — https://github.com/FarnazFouladi/HashSeq
- **R** (Runtime environment in which HashSeq is installed via devtools and inferTrueSequences/makeLoessPlot are executed) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
threshold=1000; inferTrueSequences(inputDir, outputDir, threshold)
```

## Evaluation signals

- All sequences in sequences.fasta and SvTable.txt have total reads across samples at or above the chosen threshold (e.g., >=1,000).
- Running makeLoessPlot on childrenProperties.txt shows the LOESS fit tracking the mean/SD versus parent-abundance relationship for parents above the 1,000-read threshold.
- Setting threshold=1000 on the zymo example yields cluster, children-properties, and significant-children tables consistent with the documented dimensions (e.g., 230-row children properties table, 26-row significant children table).
- Significant one-mismatch children in InferenceTestSignificantChildren.txt are attached to parents that pass the threshold, and downstream FDR-corrected P values are computable from the resulting table.

## Limitations

- Filtering at 1,000 reads makes HashSeq insensitive to true low-abundance sequences; DADA2 or similar tools are recommended when low-abundance variant detection matters.
- The threshold interacts with data-set size: the paper used 1,000 reads for all data sets except the largest one, which required 10,000 reads.
- Variants removed by the threshold cannot be evaluated by the inference test, so the filter defines the universe of callable parents.

## Evidence

- [discussion] This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000.: "This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000."
- [methods] sequence variants that have total reads less than 1000 across all samples are removed: "sequence variants that have total reads less than 1000 across all samples are removed"
- [readme] This visualization helps us to set a filtering threshold at above which the LOESS regression is a good fit for our data.: "This visualization helps us to set a filtering threshold at above which the LOESS regression is a good fit for our data."
- [discussion] our algorithm is not sensitive to detect true low-abundance sequences. Therefore, we recommend using more sensitive algorithms, such as DADA2: "our algorithm is not sensitive to detect true low-abundance sequences. Therefore, we recommend using more sensitive algorithms, such as DADA2"
