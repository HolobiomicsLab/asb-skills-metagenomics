---
name: long-read-sv-validation
description: Use when SV calls (e.g. deletions from Supernova2 assemblies or SvABA/Long
  Ranger callsets) must be independently validated against long-read sequencing data
  before benchmarking or release.
when_to_use_negative:
- Input is a small-variant (SNV) callset — benchmark those with RTG vcfeval against
  GIAB call sets instead.
- No long-read data are available for the sample; VaPoR validation cannot be performed
  and only truvari benchmarking against GIAB Tier 1 SV regions is possible.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: VaPoR
  role: Aligns SV calls to PacBio CCS long reads to validate allele sequences and
    breakpoints
- name: truvari
  role: Benchmarks SV callsets against GIAB Tier 1 SV regions to compute precision,
    recall, and F1
- name: Supernova2
  role: Produces the de novo diploid assemblies from which SVs are called directly
    and then validated
- name: SvABA
  role: Calls SVs from Illumina BWA-MEM alignments as a read-based comparison set
    for validation
- name: Long Ranger
  role: Produces deletion calls from 10x Linked-Read libraries as another read-based
    comparison set
- name: PacBio
  role: Source of CCS long reads used as orthogonal evidence for SV validation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:47b367aa9bb2f8e7cafb436138f1e5977fa138d6d5cecdd98a86b21e48c2aa85
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
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# long-read-sv-validation

## Summary

Validate assembly-derived or caller-reported structural variant calls by aligning them to PacBio CCS long reads with VaPoR, providing orthogonal long-read evidence for SV genotypes and breakpoints.

## When to use

Use when SV calls (e.g. deletions from Supernova2 assemblies or SvABA/Long Ranger callsets) must be independently validated against long-read sequencing data before benchmarking or release.

## When NOT to use

- Input is a small-variant (SNV) callset — benchmark those with RTG vcfeval against GIAB call sets instead.
- No long-read data are available for the sample; VaPoR validation cannot be performed and only truvari benchmarking against GIAB Tier 1 SV regions is possible.

## Inputs

- SV callsets in VCF format (assembly-derived from Supernova2, or read-based from SvABA / Long Ranger deletions)
- PacBio CCS long reads (FASTQ/BAM) for the same sample (e.g. NA12878, NA24385)

## Outputs

- Per-SV validation classifications from VaPoR (supported / weakly supported / unsupported)
- Per-sample concordance metrics (precision, recall, F1) for validated SV callsets, compiled into Supplementary Tables S7–S14

## How to apply

Take the SV callsets produced by assembly-based calling (e.g. from Supernova2 de novo assemblies of NA12878 and NA24385) and from read-based callers (SvABA on BWA-MEM Illumina alignments, or the Long Ranger pipeline on 10x Linked-Read libraries), and align each call to PacBio CCS long reads using VaPoR. VaPoR assesses whether the long reads support the SV allele sequence, yielding a validated subset with per-call support classifications. Compile concordance statistics per sample and caller (precision, recall, F1) alongside the truvari benchmarking against GIAB Tier 1 SV regions, and record results in the concordance tables corresponding to Supplementary Tables S7–S14. The validation step is most informative for assembly-derived calls, which lack the read-pileup evidence available to read-based callers.

## Related tools

- **VaPoR** (Aligns SV calls to PacBio CCS long reads to validate allele sequences and breakpoints)
- **truvari** (Benchmarks SV callsets against GIAB Tier 1 SV regions to compute precision, recall, and F1)
- **Supernova2** (Produces the de novo diploid assemblies from which SVs are called directly and then validated)
- **SvABA** (Calls SVs from Illumina BWA-MEM alignments as a read-based comparison set for validation)
- **Long Ranger** (Produces deletion calls from 10x Linked-Read libraries as another read-based comparison set)
- **PacBio** (Source of CCS long reads used as orthogonal evidence for SV validation)

## Examples

```
vapor -c assembly_sv.vcf -r hg38.fa -b pacbio_ccs.bam -o vapor_validated
```

## Evaluation signals

- VaPoR classifications exist for each assembly-derived SV call submitted, with a clear supported/unsupported label per call.
- Per-sample concordance statistics (precision, recall, F1) after VaPoR validation reproduce the GIAB-comparison values reported in Supplementary Tables S7–S14 for both NA12878 and NA24385.
- truvari benchmarking against GIAB Tier 1 SV regions agrees with the VaPoR-supported subset rather than the raw callset.
- Callsets are stratified by caller (assembly-based, SvABA, Long Ranger) and each stratum has independently computed validation metrics.

## Limitations

- Only SNVs are supported by the companion simulation tooling (gen_fasta.py inserts 'only SNVs for this version'), so SV validation relies entirely on real long-read data rather than simulated truth sets.
- Assembly-derived SV calls lack read-pileup support, so validation quality depends on the completeness and accuracy of the PacBio CCS read set.
- Assembly quality is sensitive to 10x library parameters (physical coverage should be kept within ∼332×–823× and not exceed 1,000× for C = 56×); poorly parameterized assemblies may yield SV calls that fail long-read validation.

## Evidence

- [methods] Validate assembly-derived SV calls by aligning them to PacBio CCS long reads with VaPoR: [evidence span withheld — anchor status: not_found]
- [methods] SV call sets benchmarked against GIAB Tier 1 SV regions: [evidence span withheld — anchor status: not_found]
- [discussion] All calls compared to the GIAB benchmark: "All calls were compared to the GIAB benchmark."
- [abstract] 10x Linked-Read long-range information supports de novo assembly and variant detection: "10x Linked-Read sequencing, with its high base quality and long-range information, has been demonstrated to facilitate de novo assembly of human genomes and variant detection."
