---
name: variant-call-validation-against-ground-truth-vcf
description: Use when a variant caller emits fasta/KisSNP2-format variant contexts
  that must be converted to VCF and benchmarked for SNP/INDEL sensitivity and precision
  against a known truth set (e.g., NCBI GIAB NA12878_HG001).
when_to_use_negative:
- Calls are already in normalized VCF coordinates against the same reference build
  as the truth set and need no fasta-to-VCF conversion or strand-duplicate removal
- No reliable ground-truth VCF exists for the sample (the article notes real-data
  evaluation relies on an approximated ground truth, lowering precision estimates)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: hap.py
  role: Benchmarks converted query VCFs against the ground-truth VCF to compute per-type
    precision/recall
  repo: https://github.com/Illumina/hap.py
- name: snp2vcf
  role: Pipeline (seqtk -> bwa-mem -> sam2vcf) that converts caller fasta outputs
    to VCF by aligning variant fragments against hg38
- name: ebwt2InDel
  role: Variant caller whose output .snp/fasta file is one query side of the validation
  repo: https://github.com/nicolaprezza/ebwt2InDel
- name: DISCOSNP++
  role: Competitor de Bruijn graph variant caller whose output is converted and benchmarked
    the same way
- name: filter_snp
  role: Post-processing tool retaining only variants covered by at least m reads (e.g.,
    5) before conversion and validation
  repo: https://github.com/nicolaprezza/ebwt2InDel
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:a00b9c668ae9b0abff18f83aa2b6c1f536e98dfbcbd202e826b272b2d8acae07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Illumina/hap.py
  tools:
  - BCR
  - DISCOSNP++
  - ebwt2InDel
  - filter_snp
  - hap.py
  - snp2vcf
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# variant-call-validation-against-ground-truth-vcf

## Summary

Benchmark reference-free variant calls (SNPs and INDELs) against a gold-standard truth VCF using hap.py, converting caller outputs to VCF first via the snp2vcf pipeline (seqtk -> bwa-mem -> sam2vcf) aligned against hg38, to obtain per-variant-type sensitivity/precision for tools like EBWT2INDEL and DISCOSNP++.

## When to use

Use when a variant caller emits fasta/KisSNP2-format variant contexts that must be converted to VCF and benchmarked for SNP/INDEL sensitivity and precision against a known truth set (e.g., NCBI GIAB NA12878_HG001).

## When NOT to use

- Calls are already in normalized VCF coordinates against the same reference build as the truth set and need no fasta-to-VCF conversion or strand-duplicate removal
- No reliable ground-truth VCF exists for the sample (the article notes real-data evaluation relies on an approximated ground truth, lowering precision estimates)

## Inputs

- Caller variant output in KisSNP2/fasta format (e.g., filtered ebwt2InDel .snp-derived fasta)
- Ground-truth VCF (NCBI GIAB NA12878_HG001 release or 1000 Genomes phase 3 20130502 VCF)
- Reference genome FASTA (hg38 Human assembly)
- High-confidence regions BED file for hap.py (e.g., confident.bed)

## Outputs

- Per-caller VCF of variant calls (post duplicate removal)
- hap.py summary of SNP and INDEL sensitivity (recall) and precision per tool
- Sensitivity comparison table (e.g., sensitivity_comparison_results.csv)

## How to apply

Convert each caller's fasta output with the snp2vcf pipeline: extract variant fragments with seqtk, align them against the hg38 Human assembly using bwa-mem, and produce VCF entries via sam2vcf. Because both tools analyze a dataset and its reverse-complement, remove duplicate VCF entries arising from forward/reverse strand detection before validation. Then run hap.py with the ground-truth VCF (e.g., NCBI GIAB NA12878_HG001 release, or the 1000 Genomes Project phase 3 VCF, release 20130502, for non-GIAB samples) as truth, the caller VCF as query, and the reference FASTA, producing per-type (SNP vs INDEL) precision and recall. Compare caller sensitivities from the hap.py summary to judge relative performance, e.g., EBWT2INDEL recovering over 5x more SNPs and over 2x more INDELs than DISCOSNP++ on merged whole-genome data.

## Related tools

- **hap.py** (Benchmarks converted query VCFs against the ground-truth VCF to compute per-type precision/recall) — https://github.com/Illumina/hap.py
- **snp2vcf** (Pipeline (seqtk -> bwa-mem -> sam2vcf) that converts caller fasta outputs to VCF by aligning variant fragments against hg38)
- **ebwt2InDel** (Variant caller whose output .snp/fasta file is one query side of the validation) — https://github.com/nicolaprezza/ebwt2InDel
- **DISCOSNP++** (Competitor de Bruijn graph variant caller whose output is converted and benchmarked the same way)
- **filter_snp** (Post-processing tool retaining only variants covered by at least m reads (e.g., 5) before conversion and validation) — https://github.com/nicolaprezza/ebwt2InDel

## Examples

```
hap.py truth.vcf query.vcf -f confident.bed -o output_prefix -r reference.fa
```

## Evaluation signals

- After strand-duplicate removal, no variant appears twice in each VCF (forward/reverse-strand detections deduplicated)
- hap.py summary.csv reports separate SNP and INDEL TP/FP/FN counts and recall/precision for each caller VCF
- Reproduced expectation: EBWT2InDel sensitivity exceeds DISCOSNP++ with >5x more SNPs and >2x more INDELs recovered on merged whole-genome data
- All query VCF records map to the hg38 coordinates used by the truth VCF (no coordinate-build mismatch in hap.py output)
- Filtered query (filter_snp with minimum read count 5) yields a consistent variant count before conversion, confirming the filter was applied

## Limitations

- Real-data evaluation relies on an approximated ground truth, lowering precision estimates for both tools
- Duplicate entries arise because both tools analyze the dataset and its reverse-complement, so deduplication is required before fair counting
- The state-of-the-art competitor is described only as 'the state-of-the-art tool based on de Bruijn graphs' in some parts of the paper, and no software version numbers are stated for either tool, complicating exact reproduction

## Evidence

- [results] hap.py usage: "we used the NCBI .vcfftp://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/release/NA12878_HG001/latest/, and validated the calls using hap.py"
- [methods] snp2vcf pipeline: "We call **snp2vcf** the pipeline **seqtk -> bwa-mem -> sam2vcf**."
- [results] hg38 alignment: [evidence span withheld — anchor status: not_found]
- [results] duplicate removal: [evidence span withheld — anchor status: not_found]
- [readme] hap.py command: [evidence span withheld — anchor status: external_doc]
- [discussion] approximate ground truth: "with lower precision for both tools due to the use of an approximated ground truth"
