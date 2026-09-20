---
name: variant-call-validation-with-happy
description: Use when two or more variant-calling outputs (VCF) must be compared against
  a trusted truth VCF to quantify SNP/INDEL sensitivity and precision, e.g. benchmarking
  a new caller against a baseline.
when_to_use_negative:
- Caller output is still in raw fragment/context format (e.g. KisSNP2 fasta from ebwt2InDel)
  and has not yet been converted to VCF via seqtk/bwa-mem/sam2vcf
- The evaluation goal is positional/allele-identity matching of somatic callsets without
  haplotype resolution — use som.py-style comparison instead
- 'Real-data ground truth is only approximate: the paper notes lower precision for
  both tools due to an approximated ground truth, so precision estimates on such data
  are not comparable to synthetic-data results'
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3245
tools:
- name: hap.py
  role: Genotype-level haplotype comparison of query VCF against truth VCF to compute
    SNP/INDEL sensitivity and precision
  repo: https://github.com/Illumina/hap.py
- name: ebwt2InDel
  role: Source of the query variant calls being validated (reference-free SNP/INDEL
    caller taking the eBWT of the read collection as input)
  repo: https://github.com/nicolaprezza/ebwt2InDel
- name: snp2vcf
  role: Upstream pipeline (seqtk -> bwa-mem -> sam2vcf) converting callers' fasta
    variant-fragment outputs to VCF before validation
- name: DISCOSNP++
  role: Baseline de Bruijn graph-based caller whose output is validated with the same
    hap.py setup for fair comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:d7f34ecb0860a91955b9874b16fa5dea22867ad889006c1a0154ad0e53d62e56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Illumina/hap.py
  tools:
  - BCR
  - DISCOSNP++
  - SimSeq
  - bwa mem
  - ebwt2InDel
  - filter_snp
  - hap.py
  - sam2vcf
  - seqtk
  - snp2vcf
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3245
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# variant-call-validation-with-happy

## Summary

Benchmark variant call sets (from ebwt2InDel/DISCOSNP++ style reference-free callers or any caller emitting VCF) against a ground-truth VCF using Illumina's hap.py, yielding SNP and INDEL sensitivity/precision figures. In the paper this step validated both call sets after conversion to VCF via the snp2vcf pipeline (seqtk -> bwa-mem -> sam2vcf).

## When to use

Use when two or more variant-calling outputs (VCF) must be compared against a trusted truth VCF to quantify SNP/INDEL sensitivity and precision, e.g. benchmarking a new caller against a state-of-the-art baseline.

## When NOT to use

- Caller output is still in raw fragment/context format (e.g. KisSNP2 fasta from ebwt2InDel) and has not yet been converted to VCF via seqtk/bwa-mem/sam2vcf
- The evaluation goal is positional/allele-identity matching of somatic callsets without haplotype resolution — use som.py-style comparison instead
- Real-data ground truth is only approximate: the paper notes lower precision for both tools due to an approximated ground truth, so precision estimates on such data are not comparable to synthetic-data results

## Inputs

- Query VCF from the caller under test (e.g. converted ebwt2InDel or DISCOSNP++ output)
- Ground-truth VCF (e.g. 1000 Genomes phase 3 heterozygous variants, or NCBI GIAB NA12878_HG001 truth set)
- Reference genome fasta (hg38 Human assembly) for haplotype construction
- Optional confident-region BED file (-f)

## Outputs

- hap.py output prefix files (summary and extended metrics) giving SNP and INDEL TP/FP/FN, sensitivity and precision
- Per-class (SNP/INDEL) comparison table, e.g. sensitivity_comparison.csv aggregating both tools

## How to apply

First convert each caller's output (e.g. the KisSNP2-format fasta fragment outputs of ebwt2InDel and DISCOSNP++) to VCF by aligning variant fragments against hg38 with bwa mem via the snp2vcf pipeline (seqtk with fake base qualities -> bwa mem -> sam2vcf), removing duplicate entries caused by forward/reverse-strand detection. Then run hap.py with the truth VCF as first argument, the query VCF as second, a confident-region BED, an output prefix, and the reference fasta, so hap.py resolves complex variant representations via graph-based haplotype comparison rather than naive per-record matching. Ground truth choices in the paper: the 1000 Genomes phase 3 heterozygous VCF for synthetic Human chromosome 1 data, and the NCBI GIAB NA12878_HG001 truth set for real data. Report sensitivity and precision stratified by SNP and INDEL classes; on the 30x synthetic chr1 dataset, expect ~83% SNP / ~72% INDEL sensitivity for ebwt2InDel versus ~71% / ~51% for DISCOSNP++.

## Related tools

- **hap.py** (Genotype-level haplotype comparison of query VCF against truth VCF to compute SNP/INDEL sensitivity and precision) — https://github.com/Illumina/hap.py
- **ebwt2InDel** (Source of the query variant calls being validated (reference-free SNP/INDEL caller taking the eBWT of the read collection as input)) — https://github.com/nicolaprezza/ebwt2InDel
- **snp2vcf** (Upstream pipeline (seqtk -> bwa-mem -> sam2vcf) converting callers' fasta variant-fragment outputs to VCF before validation)
- **DISCOSNP++** (Baseline de Bruijn graph-based caller whose output is validated with the same hap.py setup for fair comparison)

## Examples

```
hap.py truth.vcf query.vcf -f confident.bed -o output_prefix -r reference.fa
```

## Evaluation signals

- On the 30x synthetic Human chromosome 1 set, hap.py-derived SNP sensitivity for ebwt2InDel should be ~83% and INDEL sensitivity ~72%, versus ~71% and ~51% for DISCOSNP++
- Both query VCFs must have been deduplicated (forward/reverse-strand double detections removed) before hap.py, otherwise TP counts inflate
- Sensitivity and precision must be reported separately for SNP and INDEL classes, since hap.py counts complex variants into both categories via REF/ALT re-alignment
- Same truth VCF, reference fasta, and confident-region BED must be used for both competing callers for the comparison to be valid

## Limitations

- Real-data validation relies on an approximated ground truth, lowering precision estimates for both tools
- hap.py haplotype comparison requires a reference genome; representation differences (MNPs, complex variants) can shift precision/FP counts unless pre-processing (pre.py splitting/trimming) is applied consistently

## Evidence

- [results] Validation against GIAB truth with hap.py: "we used the NCBI .vcfftp://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/release/NA12878_HG001/latest/, and validated the calls using hap.py"
- [results] Upstream VCF conversion of fragment outputs: [evidence span withheld — anchor status: not_found]
- [results] Duplicate-removal precondition: [evidence span withheld — anchor status: not_found]
- [readme] hap.py CLI pattern from README: [evidence span withheld — anchor status: external_doc]
- [discussion] Approximated ground truth caveat: "with lower precision for both tools due to the use of an approximated ground truth"
- [abstract] Headline sensitivity numbers to check: "our tool is indeed able to find up to 83% of the SNPs and 72% of the existing INDELs"
