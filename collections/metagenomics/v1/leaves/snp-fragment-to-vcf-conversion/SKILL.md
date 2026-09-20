---
name: snp-fragment-to-vcf-conversion
description: Use when a reference-free variant caller (e.g. ebwt2InDel, DISCOSNP++)
  emits fasta variant fragments in KisSNP2 format and you need .vcf files to validate
  calls with hap.py against a ground-truth VCF.
when_to_use_negative:
- The variant caller already emits valid, coordinate-anchored VCFs — the seqtk ->
  bwa-mem -> sam2vcf alignment step is only needed for fragment/context outputs
- No suitable reference genome is available for the target species — the pipeline's
  alignment step against hg38 cannot be replaced and reference-free callers are preferable
  for the whole analysis
- The input is raw reads rather than post-variant-discovery fasta fragments — reads
  must first be processed by ebwt2InDel/BCR or DISCOSNP++, not converted directly
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: seqtk
  role: First stage of the snp2vcf pipeline; converts/prepares fasta variant fragments
    (with fake base qualities) for bwa mem alignment
- name: bwa mem
  role: Aligns the variant-fragment fasta sequences against the hg38 Human assembly
    so fragments get chromosomal coordinates
- name: sam2vcf
  role: Third stage of the snp2vcf pipeline; converts the bwa mem SAM alignments of
    variant fragments into .vcf files
- name: hap.py
  role: Validates the converted VCF call sets against a gold-standard truth VCF, producing
    sensitivity/precision for SNPs and INDELs
  repo: https://github.com/Illumina/hap.py
- name: ebwt2InDel
  role: Produces the upstream KisSNP2-format fasta variant fragments (optionally filtered
    by filter_snp) that this pipeline converts to VCF
  repo: https://github.com/nicolaprezza/ebwt2InDel
- name: filter_snp
  role: Post-processing step that filters the .snp/.fasta output of ebwt2InDel (e.g.
    keep variants covered by at least 5 reads) before VCF conversion
  repo: https://github.com/nicolaprezza/ebwt2InDel
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:8d2ba6e3bce32c7d7490fe6d6f157e9a483d3101d62d151580e836a4d508741c
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# snp-fragment-to-vcf-conversion

## Summary

Converts reference-free variant-discovery outputs (KisSNP2-format fasta variant fragments from tools like ebwt2InDel or DISCOSNP++) into VCF files via the snp2vcf pipeline (seqtk -> bwa-mem -> sam2vcf), aligning fragments against the hg38 Human assembly so that hap.py can benchmark the calls.

## When to use

Use when a reference-free variant caller (e.g. ebwt2InDel, DISCOSNP++) emits fasta variant fragments in KisSNP2 format and you need .vcf files to validate calls with hap.py against a ground-truth VCF.

## When NOT to use

- The variant caller already emits valid, coordinate-anchored VCFs — the seqtk -> bwa-mem -> sam2vcf alignment step is only needed for fragment/context outputs
- No suitable reference genome is available for the target species — the pipeline's alignment step against hg38 cannot be replaced and reference-free callers are preferable for the whole analysis
- The input is raw reads rather than post-variant-discovery fasta fragments — reads must first be processed by ebwt2InDel/BCR or DISCOSNP++, not converted directly

## Inputs

- KisSNP2-format fasta file of variant fragments/contexts from ebwt2InDel (optionally pre-filtered with filter_snp, e.g. events supported by at least 5 reads)
- Variant-fragment fasta output from DISCOSNP++ for the same read set
- hg38 Human assembly reference FASTA (plus indices for bwa mem)
- Ground-truth variant VCF (e.g. 1000 Genomes phase 3 release 20130502 heterozygous variants or NCBI GIAB NA12878_HG001)

## Outputs

- Deduplicated .vcf file per tool per dataset (variants placed on hg38 coordinates)
- hap.py benchmarking report with SNP and INDEL sensitivity and precision (e.g. written to sensitivity_comparison.csv)

## How to apply

Run the snp2vcf pipeline: use seqtk to prepare the variant-fragment fasta (including simulated/fake base qualities) for bwa mem, then align the fragments against the hg38 Human assembly with bwa mem, and run sam2vcf on the resulting alignments to emit .vcf files. After conversion, remove duplicate entries from the .vcf files because both tools analyze the dataset and its reverse-complement, so some variants are found twice (once on each strand). Finally feed the VCFs, the hg38 reference, and the ground-truth VCF (e.g. the 1000 Genomes phase 3 heterozygous variants for chromosome 1) to hap.py to obtain SNP and INDEL sensitivity/precision. Rationale: ebwt2InDel outputs contexts surrounding SNPs/INDELs rather than coordinates, so alignment to hg38 is required to place them into standard VCF representation for fair comparison.

## Related tools

- **seqtk** (First stage of the snp2vcf pipeline; converts/prepares fasta variant fragments (with fake base qualities) for bwa mem alignment)
- **bwa mem** (Aligns the variant-fragment fasta sequences against the hg38 Human assembly so fragments get chromosomal coordinates)
- **sam2vcf** (Third stage of the snp2vcf pipeline; converts the bwa mem SAM alignments of variant fragments into .vcf files)
- **hap.py** (Validates the converted VCF call sets against a gold-standard truth VCF, producing sensitivity/precision for SNPs and INDELs) — https://github.com/Illumina/hap.py
- **ebwt2InDel** (Produces the upstream KisSNP2-format fasta variant fragments (optionally filtered by filter_snp) that this pipeline converts to VCF) — https://github.com/nicolaprezza/ebwt2InDel
- **filter_snp** (Post-processing step that filters the .snp/.fasta output of ebwt2InDel (e.g. keep variants covered by at least 5 reads) before VCF conversion) — https://github.com/nicolaprezza/ebwt2InDel

## Evaluation signals

- Each converted .vcf parses cleanly and its variants carry hg38 chromosomal coordinates and REF/ALT alleles consistent with the source fragment contexts
- Duplicate entries caused by forward/reverse-strand detection (both tools analyze the dataset and its reverse-complement) are removed before benchmarking
- hap.py on ebwt2InDel calls reaches ~83% SNP sensitivity and ~72% INDEL sensitivity versus ~71% and ~51% for DISCOSNP++ on the 30x synthetic Human chromosome 1 set
- Pre-filtering with filter_snp (m=5 reads) yields a variant set whose call counts are consistent with the threshold applied
- The same conversion pipeline (seqtk -> bwa-mem -> sam2vcf) is applied to both tools' outputs so the hap.py comparison is fair

## Limitations

- On real-data evaluations, precision is lower for both tools due to the use of an approximated ground truth, so benchmarking numbers from converted VCFs can be depressed
- Converting fragments by alignment against hg38 introduces reference-genome dependence into an otherwise reference-free workflow, which can bias placement of variants in divergent regions
- Duplicate variant entries arise because both tools analyze the dataset and its reverse-complement; failure to deduplicate will inflate call counts before hap.py comparison

## Evidence

- [methods] snp2vcf pipeline defined in methods: "We call **snp2vcf** the pipeline **seqtk -> bwa-mem -> sam2vcf**."
- [results] alignment against hg38: [evidence span withheld — anchor status: not_found]
- [results] duplicate removal after strand analysis: [evidence span withheld — anchor status: not_found]
- [results] validation with hap.py: "and validated the calls using hap.py"
- [readme] hap.py benchmarking purpose from README: [evidence span withheld — anchor status: not_found]
