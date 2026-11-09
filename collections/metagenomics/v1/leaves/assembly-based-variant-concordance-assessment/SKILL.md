---
name: assembly-based-variant-concordance-assessment
description: Use when you have de novo diploid assemblies (e.g. Supernova2 output
  from 10x Linked-Read data) and need to quantify how accurately their variant calls
  agree with reference-based call sets and GIAB benchmarks.
when_to_use_negative:
- Input is simulated Linked-Read data being generated or parameter-tuned (use the
  LRTK-SIM simulation/assembly-quality workflow instead of variant concordance benchmarking)
- No GIAB benchmark call sets or trusted truth sets exist for your sample, since vcfeval/truvari
  concordance scoring requires a benchmark to compare against
- The goal is assembly contiguity assessment (N50/NA50, phase blocks) rather than
  variant-call accuracy evaluation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BWA-MEM
  role: Aligns NA12878/NA24385 Illumina short reads to the hg38 reference for reference-based
    calling
- name: FreeBayes
  role: Calls SNVs from the Illumina alignments
- name: SvABA
  role: Calls SVs from the same Illumina alignments
- name: Long Ranger
  role: Processes 10x Linked-Read libraries to obtain SNV calls and deletion calls
  repo: https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger
- name: Supernova2
  role: Produces the de novo diploid assemblies from which variants are called directly
    (assembly-based calling)
- name: RTG vcfeval
  role: Benchmarks SNV call sets against GIAB benchmarks using genotype-aware matching
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
  source_content_hash: sha256:e9a8117d83ed01b77e22f2911568c97ab043ed65d3cf2e7e7c84566a3253ceff
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# assembly-based-variant-concordance-assessment

## Summary

Benchmarks variant calls derived from de novo diploid assemblies against calls from standard reference-based pipelines (Illumina and 10x Linked-Read), quantifying precision, recall, and F1 against GIAB benchmark call sets. It answers whether assembly-derived SNV/SV calls can match reference-mapping-based calling accuracy.

## When to use

Use when you have de novo diploid assemblies (e.g. Supernova2 output from 10x Linked-Read data) and need to quantify how accurately their variant calls agree with reference-based call sets and GIAB benchmarks.

## When NOT to use

- Input is simulated Linked-Read data being generated or parameter-tuned (use the LRTK-SIM simulation/assembly-quality workflow instead of variant concordance benchmarking)
- No GIAB benchmark call sets or trusted truth sets exist for your sample, since vcfeval/truvari concordance scoring requires a benchmark to compare against
- The goal is assembly contiguity assessment (N50/NA50, phase blocks) rather than variant-call accuracy evaluation

## Inputs

- Supernova2 de novo diploid assemblies from 10x Linked-Read data (NA12878, NA24385)
- Standard Illumina short reads aligned to hg38 via BWA-MEM (BAM)
- 10x Linked-Read libraries processed through Long Ranger
- GIAB benchmark call sets and GIAB Tier 1 SV regions
- PacBio CCS long reads for SV validation

## Outputs

- Per-sample SNV/SV concordance metrics (precision, recall, F1) by callset and caller
- vcfeval genotype-aware comparison results against GIAB benchmark call sets
- truvari comparison results against GIAB Tier 1 SV regions
- VaPoR validation assessments of assembly-derived SV calls
- Concordance tables (corresponding to Supplementary Tables S7–S14)

## How to apply

Align Illumina short reads (NA12878, NA24385) to the hg38 reference with BWA-MEM, then call SNVs with FreeBayes and SVs with SvABA; process 10x Linked-Read libraries with the Long Ranger pipeline to obtain SNV and deletion calls; call variants directly from the Supernova2 de novo assemblies. Benchmark the resulting SNV call sets against GIAB benchmark call sets with RTG vcfeval using genotype-aware matching, and SV call sets against GIAB Tier 1 SV regions with truvari. Validate assembly-derived SV calls by aligning them to PacBio CCS long reads with VaPoR, then compile per-sample precision, recall, and F1 by callset and caller into concordance tables.

## Related tools

- **BWA-MEM** (Aligns NA12878/NA24385 Illumina short reads to the hg38 reference for reference-based calling)
- **FreeBayes** (Calls SNVs from the Illumina alignments)
- **SvABA** (Calls SVs from the same Illumina alignments)
- **Long Ranger** (Processes 10x Linked-Read libraries to obtain SNV calls and deletion calls) — https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger
- **Supernova2** (Produces the de novo diploid assemblies from which variants are called directly (assembly-based calling))
- **RTG vcfeval** (Benchmarks SNV call sets against GIAB benchmarks using genotype-aware matching)
- **truvari** (Benchmarks SV call sets against GIAB Tier 1 SV regions)
- **VaPoR** (Validates assembly-derived SV calls by aligning them to PacBio CCS long reads)

## Evaluation signals

- vcfeval concordance statistics reproduce the GIAB-comparison values reported in Tables S7–S14 for NA12878
- vcfeval concordance statistics reproduce the GIAB-comparison values reported in Tables S7–S14 for NA24385
- truvari comparisons against GIAB Tier 1 SV regions reproduce the reported SV concordance metrics
- VaPoR validates assembly-derived SV calls against PacBio CCS long reads with agreement consistent with reported validation
- Expected directional result: SNV calls from reference-based processing of Illumina and 10x data are comparable in accuracy, and both exceed the accuracy of SNV calls derived from the de novo assemblies

## Limitations

- Assembly-based SNV calls are less accurate than reference-based SNV calls from standard Illumina and 10x data
- No alternative assembler to Supernova2 was available for comparison, limiting interpretation of assembler-specific effects
- All calls are compared to the GIAB benchmark, so accuracy is bounded by the coverage and quality of the benchmark regions (e.g. GIAB Tier 1 SV regions)

## Evidence

- [discussion] GIAB benchmark comparison: "All calls were compared to the GIAB benchmark."
- [intro] de novo assembly toolchain: "These simulated and real data sets were then used to produce de novo assemblies, with an emphasis on the performance of 10x's Supernova2 [24]."
- [abstract] truth samples: "We prepared and sequenced eight 10x libraries with a diverse set of parameters from standard cell lines NA12878 and NA24385"
- [readme] Long Ranger compatibility: "The simulated data have been tested and accepted by 10X official pipelines [Long Ranger](https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger) and"
