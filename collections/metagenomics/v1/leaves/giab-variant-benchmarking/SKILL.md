---
name: giab-variant-benchmarking
description: Use when you have variant calls (VCF) from multiple strategies — e.g.
  FreeBayes/SvABA on Illumina BAMs, Long Ranger on 10x Linked-Reads, or calling directly
  from Supernova2 assemblies — and need to score them against GIAB truth sets.
when_to_use_negative:
- Input is simulated data with a known truth set (e.g. LRTK-SIM-generated Linked-Reads
  from gen_fasta.py haplotypes) — compare directly to the simulated truth instead
  of GIAB.
- The goal is assembly contiguity/quality assessment (N50, NA50) rather than variant-call
  accuracy.
- The sample has no adequate GIAB benchmark call set or Tier 1 SV regions available.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: BWA-MEM
  role: Aligns NA12878/NA24385 Illumina short reads to the hg38 reference to produce
    the BAM input for FreeBayes and SvABA
- name: FreeBayes
  role: Calls SNVs from the Illumina alignments for the reference-based call set
- name: SvABA
  role: Calls SVs from the same Illumina alignments for the reference-based call set
- name: Long Ranger
  role: 10x official pipeline that produces SNV and deletion calls from the 10x Linked-Read
    libraries
  repo: https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger
- name: Supernova2
  role: Produces the de novo diploid assemblies from which assembly-based SNV and
    SV calls are derived
- name: RTG vcfeval
  role: Benchmarks SNV call sets against GIAB benchmark call sets using genotype-aware
    matching
- name: truvari
  role: Benchmarks SV call sets against GIAB Tier 1 SV regions
- name: VaPoR
  role: Validates assembly-derived SV calls by aligning them to PacBio CCS long reads
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:edeaada887908d1e7f540209e2a7cd48e7087cf646fb801c74950e29b4818dc5
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
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# giab-variant-benchmarking

## Summary

Benchmark variant call sets (SNVs and SVs) from reference-based Illumina/10x pipelines and de novo assemblies against GIAB benchmark call sets, quantifying precision, recall, and F1 per caller and sample. This provides the ground-truth comparison that in the source study showed reference-based SNV calls outperform assembly-derived SNV calls.

## When to use

Use when you have variant calls (VCF) from multiple strategies — e.g. FreeBayes/SvABA on Illumina BAMs, Long Ranger on 10x Linked-Reads, or calling directly from Supernova2 assemblies — and need to score them against GIAB truth sets.

## When NOT to use

- Input is simulated data with a known truth set (e.g. LRTK-SIM-generated Linked-Reads from gen_fasta.py haplotypes) — compare directly to the simulated truth instead of GIAB.
- The goal is assembly contiguity/quality assessment (N50, NA50) rather than variant-call accuracy.
- The sample has no adequate GIAB benchmark call set or Tier 1 SV regions available.

## Inputs

- Illumina short-read FASTQ for GIAB samples (NA12878, NA24385)
- 10x Linked-Read FASTQ libraries
- hg38 human reference sequence
- Supernova2 de novo diploid assembly outputs
- GIAB benchmark VCF call sets and GIAB Tier 1 SV region BED

## Outputs

- VCF call sets per caller and sample (FreeBayes SNVs, SvABA SVs, Long Ranger SNV/deletion calls, assembly-derived SNV/SV calls)
- RTG vcfeval concordance output (TP/FP/FN with genotype-aware matching)
- truvari comparison results for SVs against GIAB Tier 1 SV regions
- VaPoR validation results for assembly-derived SV calls
- Per-sample precision, recall, and F1 tables by callset and caller (cf. Supplementary Tables S7–S14)

## How to apply

First produce the call sets to be compared: align NA12878/NA24385 Illumina short reads to the hg38 reference with BWA-MEM, call SNVs with FreeBayes and SVs with SvABA from the same alignments, run the 10x Long Ranger pipeline on the Linked-Read libraries for SNV and deletion calls, and call variants directly from Supernova2 de novo diploid assemblies. Then benchmark SNV call sets against the GIAB benchmark call sets with RTG vcfeval using genotype-aware matching, and benchmark SV call sets against the GIAB Tier 1 SV regions with truvari. For assembly-derived SV calls, additionally validate by aligning the SVs to PacBio CCS long reads with VaPoR, since GIAB SV truth is incomplete. Compile per-sample precision, recall, and F1 by callset and caller into concordance tables (corresponding to Supplementary Tables S7–S14) for both NA12878 and NA24385.

## Related tools

- **BWA-MEM** (Aligns NA12878/NA24385 Illumina short reads to the hg38 reference to produce the BAM input for FreeBayes and SvABA)
- **FreeBayes** (Calls SNVs from the Illumina alignments for the reference-based call set)
- **SvABA** (Calls SVs from the same Illumina alignments for the reference-based call set)
- **Long Ranger** (10x official pipeline that produces SNV and deletion calls from the 10x Linked-Read libraries) — https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger
- **Supernova2** (Produces the de novo diploid assemblies from which assembly-based SNV and SV calls are derived)
- **RTG vcfeval** (Benchmarks SNV call sets against GIAB benchmark call sets using genotype-aware matching)
- **truvari** (Benchmarks SV call sets against GIAB Tier 1 SV regions)
- **VaPoR** (Validates assembly-derived SV calls by aligning them to PacBio CCS long reads)

## Examples

```
rtg vcfeval -b NA12878_GIAB.vcf.gz -c FreeBayes_NA12878.vcf.gz -o vcfeval_NA12878_out
```

## Evaluation signals

- vcfeval, truvari, and VaPoR concordance statistics reproduce the GIAB-comparison values reported in Supplementary Tables S7–S14 for both NA12878 and NA24385.
- SNV benchmarking uses genotype-aware matching (vcfeval), not position-only overlap, so genotype mismatches count as discordant.
- SV benchmarking is restricted to GIAB Tier 1 SV regions rather than the whole genome.
- Expected qualitative outcome: reference-based Illumina and 10x SNV calls have comparable accuracy, and both exceed the accuracy of SNV calls derived from the Supernova2 de novo assemblies.
- Every assembly-derived SV that contributes to precision should be VaPoR-validated against PacBio CCS reads.

## Limitations

- SNV calls derived from de novo diploid assemblies were less accurate than SNV calls from reference-based processing of either standard Illumina or 10x Linked-Read data.
- Assembly-derived SV calls require supplementary validation (VaPoR against PacBio CCS) because GIAB SV truth is incomplete.
- No alternative assembler to Supernova2 was available for comparison, limiting generalization of results to other assembly pipelines.

## Evidence

- [discussion] All calls were compared to the GIAB benchmark.: "All calls were compared to the GIAB benchmark."
- [intro] 10x Linked-Read enables de novo assembly and variant detection: "10x Linked-Read sequencing, with its high base quality and long-range information, has been demonstrated to facilitate de novo assembly of human genomes and variant detection."
- [abstract] Standard cell lines used for benchmarking: "We prepared and sequenced eight 10x libraries with a diverse set of parameters from standard cell lines NA12878 and NA24385"
