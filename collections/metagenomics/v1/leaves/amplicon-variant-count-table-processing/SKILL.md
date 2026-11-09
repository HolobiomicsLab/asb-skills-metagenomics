---
name: amplicon-variant-count-table-processing
description: Use when raw pre-processed 16S rRNA amplicon FASTQ data (e.g., PRJEB24409
  mock communities with primers removed, reads paired, trimmed to 250 bases) must
  be converted into a filtered variant count table for downstream analyses.
when_to_use_negative:
- The goal is to detect true low-abundance sequence variants — HashSeq is conservative
  and not sensitive for low-abundance sequences; the article recommends more sensitive
  algorithms such as DADA2
- Variants with two or more mismatches, indel variation relative to a parent, or long-read
  data such as PacBio are of interest — HashSeq considers only one-mismatch children
  and does not explicitly model indels
- Input is already a filtered count/feature table (e.g., an existing SvTable.txt);
  re-running variant calling would be redundant
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: HashSeq
  role: Infers true biological 16S rRNA sequence variants from Illumina errors via
    HashMap-based one-mismatch clustering and normal-distribution/LOESS background
    error estimation, producing SvTable.txt and sequences.fasta
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Alternative variant caller that models error rate as a function of quality
    scores per nucleotide transition; used for comparison of count tables in association
    studies and recommended for low-abundance variants
- name: scikit-learn
  role: Trains random forest classifiers (100 trees) on the caller variant tables
    merged with metadata, evaluated with RepeatedKFold (4 splits x 10 repeats)
- name: R
  role: Language in which HashSeq is distributed as a package and in which the downstream
    table processing and figure scripts are written
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:2ade3432b459cc5093afbe617eb3116b000153ae1f274a4fc771b84ad2e0e93a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - Deblur
  - HashSeq
  - R
  - Unoise2
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# amplicon-variant-count-table-processing

## Summary

Process 16S rRNA amplicon sequencing data into filtered sequence-variant count tables by calling variants with HashSeq's inferTrueSequences (HashMap-based clustering plus normal-distribution/LOESS error modeling) and removing variants below an abundance threshold (1,000 reads total; 10,000 for the largest data set), yielding SvTable.txt/sequences.fasta suitable for downstream BLAST identity checks and supervised classification.

## When to use

Use when raw pre-processed 16S rRNA amplicon FASTQ data (e.g., PRJEB24409 mock communities with primers removed, reads paired, trimmed to 250 bases) must be converted into a filtered variant count table for downstream analyses.

## When NOT to use

- The goal is to detect true low-abundance sequence variants — HashSeq is conservative and not sensitive for low-abundance sequences; the article recommends more sensitive algorithms such as DADA2
- Variants with two or more mismatches, indel variation relative to a parent, or long-read data such as PacBio are of interest — HashSeq considers only one-mismatch children and does not explicitly model indels
- Input is already a filtered count/feature table (e.g., an existing SvTable.txt); re-running variant calling would be redundant

## Inputs

- Pre-processed 16S rRNA gene fastq files (primers removed, forward/reverse reads paired, trimmed to 250 bases), e.g., PRJEB24409 mock-community data from github.com/FarnazFouladi/Examples
- Parent-abundance filtering threshold (default 1,000 total reads; 10,000 for the largest data set)
- Sample metadata table for merging with the variant count table (e.g., autism gut data set PRJNA533120, n = 81)

## Outputs

- SvTable.txt: filtered sequence-variant count table for taxonomic assignment and downstream analyses
- sequences.fasta: parent sequences above the threshold plus significant one-mismatch children
- OneMismatchCluster.txt, childrenProperties.txt, InferenceTestSignificantChildren.txt (intermediate HashSeq outputs)
- rf_cv_performance.csv: per-caller, per-variable random forest cross-validated performance (Fig. 7-style comparison)
- silva_identity_table.csv and Fig. 6-style summary: per-variant caller, top SILVA132 hit accession, and percent identity

## How to apply

First pre-process the fastq data by removing primers, pairing forward and reverse reads, and trimming sequences to 250 bases, then call variants with HashSeq's inferTrueSequences(inputDir, outputDir, threshold), which clusters one-mismatch children with a HashMap-based approach and estimates background error rates with a normal distribution combined with LOESS regression as a function of sequencing depth. Set the parent-abundance threshold to 1,000 reads total across all samples (10,000 for the largest data set); variants below the threshold are removed, and one-mismatch children surviving an inference test are retained alongside parents. Inspect childrenProperties.txt with makeLoessPlot to confirm the LOESS regression is a good fit above the chosen threshold before finalizing filter settings. The resulting SvTable.txt count table can then be merged with sample metadata for downstream tasks such as random forest classification (100 trees, RepeatedKFold 4x10 cross-validation) or BLAST alignment of sequences.fasta against SILVA132 to compute percent identity. For large data sets, raise the JVM heap via options(java.parameters) after restarting R and before attaching libraries.

## Related tools

- **HashSeq** (Infers true biological 16S rRNA sequence variants from Illumina errors via HashMap-based one-mismatch clustering and normal-distribution/LOESS background error estimation, producing SvTable.txt and sequences.fasta) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Alternative variant caller that models error rate as a function of quality scores per nucleotide transition; used for comparison of count tables in association studies and recommended for low-abundance variants)
- **scikit-learn** (Trains random forest classifiers (100 trees) on the caller variant tables merged with metadata, evaluated with RepeatedKFold (4 splits x 10 repeats))
- **R** (Language in which HashSeq is distributed as a package and in which the downstream table processing and figure scripts are written) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
options(java.parameters="-Xmx1024m"); library(HashSeq); inferTrueSequences(inputDir, outputDir, threshold=1000)
```

## Evaluation signals

- SvTable.txt contains only variants whose total reads across all samples meet or exceed the threshold (1,000, or 10,000 for the largest data set)
- The saved rf_cv_performance.csv contains one performance row per caller-variable combination, and HashSeq vs. DADA2 performance values are near-identical as reported in Fig. 7
- The silva_identity_table.csv shows the great majority of HashSeq variants with >99% percent identity to their top SILVA132 hit, matching Table 1 / Fig. 6
- makeLoessPlot on childrenProperties.txt shows the LOESS regression fits the mean/standard deviation of one-mismatch children vs. parent abundance above the chosen threshold
- HashSeq-derived variant sets are conservative: fewer sequence variants are called as true compared to DADA2

## Limitations

- HashSeq is not sensitive for detecting true low-abundance sequences; use more sensitive algorithms such as DADA2 for low-abundance variants
- The algorithm considers only one-mismatch variants and does not consider two or more mismatches
- Indels are not explicitly modeled and are treated as a separate parent sequence; for indel variation relative to a parent, Deblur or multiple-sequence-alignment methods are suggested
- The rarity assumption for multiple-mismatch sequences may not hold for long-read technologies such as PacBio
- Compositionality and compositional artifacts are not addressed
- Default JVM heap size (512 MB) may be insufficient for large data sets; options(java.parameters) must be set after restarting R and before attaching libraries

## Evidence

- [discussion] This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000.: "This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000."
- [readme] SvTable.txt count table used downstream: "SvTable.txt: This count table includes all sequences from step 4. This table can be used for taxonomic assignment and other down-stream analyses."
- [abstract] Normal distribution plus LOESS error estimation: "use the normal distribution combined with locally estimated scatterplot smoothing (LOESS) regression to estimate background error rates as a function of sequencing depth for individual clusters"
- [discussion] FDR correction downstream: "which can be fed into statistical models to produce FDR-corrected P values"
- [readme] Threshold removes low-total-read variants: "Here we choose a threshold of 1000 which means sequence variants that have total reads less than 1000 across all samples are removed."
