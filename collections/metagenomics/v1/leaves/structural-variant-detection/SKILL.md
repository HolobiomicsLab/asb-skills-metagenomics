---
name: structural-variant-detection
description: Decision support for choosing between reference-based and assembly-based
  variant calling when NA12878/NA24385-style Illumina or 10x Linked-Read data are
  available and GIAB truth sets can benchmark accuracy.
when_to_use_negative:
- When no GIAB benchmark call set or Tier 1 SV truth regions exist for the sample,
  so vcfeval/truvari concordance cannot be computed
- When the goal is assembly contiguity/quality assessment (N50, NA50, phase blocks)
  rather than variant-call accuracy benchmarking — the article's LRTK-SIM/Evaluate_diploid_assembly
  skill covers that instead
- When only PacBio or ONT long-read data are available, since this skill is built
  around Illumina short-read and 10x Linked-Read inputs with Supernova2 assemblies
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: BWA-MEM
  role: Aligns NA12878 and NA24385 Illumina short reads to the hg38 reference before
    FreeBayes/SvABA calling
- name: FreeBayes
  role: Calls SNVs from the Illumina hg38 alignments
- name: SvABA
  role: Calls SVs from the same Illumina alignments
- name: Long Ranger
  role: 10x pipeline producing SNV and deletion calls from Linked-Read libraries
  repo: https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger
- name: Supernova2
  role: Produces the de novo diploid assemblies from which variants are called directly
    (assembly-based calling)
  repo: https://support.10xgenomics.com/de-novo-assembly/software/overview/welcome
- name: RTG vcfeval
  role: Genotype-aware benchmarking of SNV call sets against GIAB truth
- name: truvari
  role: Benchmarks SV call sets against GIAB Tier 1 SV regions
- name: VaPoR
  role: Validates assembly-derived SV calls by alignment to PacBio CCS long reads
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:0ae4d4a010cef3dd17cde73c5975c7e9affa35c23b98dbecc3a2c9f54812562a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA-MEM
  - FreeBayes
  - Long Ranger
  - PacBio
  - RTG vcfeval
  - Supernova2
  - SvABA
  - VaPoR
  - truvari
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# structural-variant-detection

## Summary

A benchmarking skill for detecting and validating SNV and SV calls from reference-based alignments (Illumina BWA-MEM + FreeBayes/SvABA; 10x Long Ranger) versus variant calling directly from Supernova2 de novo diploid assemblies, scored against GIAB benchmark truth sets.

## When to use

Decision support for choosing between reference-based and assembly-based variant calling when NA12878/NA24385-style Illumina or 10x Linked-Read data are available and GIAB truth sets can benchmark accuracy.

## When NOT to use

- When no GIAB benchmark call set or Tier 1 SV truth regions exist for the sample, so vcfeval/truvari concordance cannot be computed
- When the goal is assembly contiguity/quality assessment (N50, NA50, phase blocks) rather than variant-call accuracy benchmarking — the article's LRTK-SIM/Evaluate_diploid_assembly skill covers that instead
- When only PacBio or ONT long-read data are available, since this skill is built around Illumina short-read and 10x Linked-Read inputs with Supernova2 assemblies

## Inputs

- Illumina short-read FASTQ files for NA12878 and NA24385
- 10x Linked-Read FASTQ libraries from NA12878 and NA24385 cell lines
- hg38 human reference sequence
- Supernova2 de novo diploid assembly outputs
- GIAB benchmark SNV call sets and GIAB Tier 1 SV regions
- PacBio CCS long reads for SV validation

## Outputs

- Per-sample SNV concordance metrics (precision, recall, F1) from RTG vcfeval
- SV concordance metrics from truvari against GIAB Tier 1 SV regions
- VaPoR validation results for assembly-derived SV calls
- Concordance tables corresponding to Supplementary Tables S7–S14

## How to apply

Align Illumina short reads to hg38 with BWA-MEM, then call SNVs with FreeBayes and SVs with SvABA from the same alignments; process 10x Linked-Read libraries through the Long Ranger pipeline for SNV and deletion calls. In parallel, call SNVs and SVs directly from Supernova2 de novo diploid assemblies (assembly-based calling). Benchmark SNV call sets against GIAB benchmark call sets with RTG vcfeval using genotype-aware matching, and SV call sets against GIAB Tier 1 SV regions with truvari; validate assembly-derived SV calls by aligning them to PacBio CCS long reads with VaPoR. Compile per-sample precision, recall, and F1 by callset and caller into concordance tables (Supplementary Tables S7–S14). The expected outcome is that reference-based SNV calls from Illumina and 10x data have comparable accuracy, and both outperform SNVs derived from de novo assemblies.

## Related tools

- **BWA-MEM** (Aligns NA12878 and NA24385 Illumina short reads to the hg38 reference before FreeBayes/SvABA calling)
- **FreeBayes** (Calls SNVs from the Illumina hg38 alignments)
- **SvABA** (Calls SVs from the same Illumina alignments)
- **Long Ranger** (10x pipeline producing SNV and deletion calls from Linked-Read libraries) — https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger
- **Supernova2** (Produces the de novo diploid assemblies from which variants are called directly (assembly-based calling)) — https://support.10xgenomics.com/de-novo-assembly/software/overview/welcome
- **RTG vcfeval** (Genotype-aware benchmarking of SNV call sets against GIAB truth)
- **truvari** (Benchmarks SV call sets against GIAB Tier 1 SV regions)
- **VaPoR** (Validates assembly-derived SV calls by alignment to PacBio CCS long reads)

## Examples

```
python LRTK-SIM.py ./diploid_config  # generate 10x FASTQ, then: bwa mem hg38 reads | FreeBayes / Long Ranger / Supernova2 calling, benchmarked via rtg vcfeval and truvari
```

## Evaluation signals

- vcfeval, truvari, and VaPoR concordance statistics reproduce the GIAB-comparison values reported in Tables S7–S14 for both NA12878 and NA24385
- Per-callset precision, recall, and F1 are compiled for every caller (FreeBayes, SvABA, Long Ranger, assembly-based calling) and sample
- Reference-based SNV calls (Illumina and 10x) should show comparable accuracy and higher concordance than assembly-derived SNV calls
- All calls must be comparable against the GIAB benchmark, as stated in the article's methods

## Limitations

- Supernova2 de novo assemblies yield SNV calls less accurate than reference-based approaches, so assembly-based calling should not be treated as the accuracy leader
- Only SNVs are supported by the auxiliary genotyping pipeline in this version of the ecosystem (gen_fasta.py inserts 'only SNVs for this version')
- No alternative assembler to Supernova2 was available for comparison, limiting interpretation of assembly-quality effects on variant calling

## Evidence

- [discussion] All calls were compared to the GIAB benchmark.: "All calls were compared to the GIAB benchmark."
- [intro] These simulated and real data sets were then used to produce de novo assemblies, with an emphasis on the performance of 10x's Supernova2 [24].: "These simulated and real data sets were then used to produce de novo assemblies, with an emphasis on the performance of 10x's Supernova2 [24]."
- [readme] The simulated data have been tested and accepted by 10X official pipelines [Long Ranger] and [Supernova].: "The simulated data have been tested and accepted by 10X official pipelines [Long Ranger] and [Supernova]."
- [methods] gen_fasta.py can generate two haploid fasta files with variants from vcf and reference genome sequence.: "`gen_fasta.py` can generate two haploid fasta files with variants from vcf and reference genome sequence."
