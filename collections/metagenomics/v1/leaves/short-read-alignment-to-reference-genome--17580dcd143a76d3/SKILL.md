---
name: short-read-alignment-to-reference-genome
description: Use when you have reference-free variant calls (KisSNP2-format fasta
  / .snp file from ebwt2InDel) and an available reference genome, and you need reference-coordinate
  .vcf entries for downstream validation or comparison.
when_to_use_negative:
- No reference genome is available for the organism — ebwt2InDel is designed precisely
  for reference-free discovery, and alignment cannot place calls in reference coordinates
- Input reads contain characters other than A/C/G/T (e.g. 'N') — reads must be filtered
  or N replaced/removed before the upstream ebwt2InDel step
- You need genotypes/haplotypes rather than variant positions — this pipeline only
  maps mismatch/INDEL records to reference coordinates
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: filter_snp
  role: Filters the ebwt2InDel .snp file to keep only SNP/INDELs covered by at least
    m reads before conversion
  repo: github.com/nicolaprezza/ebwt2InDel
- name: seqtk
  role: Converts the filtered call file to FASTQ with fake base qualities (seqtk seq
    -F 'I')
- name: bwa mem
  role: Aligns the variant-fragment FASTQ against the hg38 Human reference assembly,
    yielding a SAM file
- name: sam2vcf
  role: Translates the SAM's mismatch and INDEL records into VCF entries
  repo: github.com/nicolaprezza/ebwt2InDel
- name: hap.py
  role: Optionally validates the resulting .vcf calls against a gold-standard truth
    set (NCBI GIAB NA12878_HG001)
  repo: Illumina/hap.py
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:a61c1247050e02249686113adb902e1590770f730244495f2a00805ef9d3fc97
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bwa mem
  - ebwt2InDel
  - filter_snp
  - hap.py
  - sam2vcf
  - seqtk
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# short-read-alignment-to-reference-genome

## Summary

Converts reference-free SNP/INDEL calls from ebwt2InDel into VCF format by aligning variant-context fragments against the hg38 Human assembly via the snp2vcf pipeline (seqtk -> bwa-mem -> sam2vcf).

## When to use

Use when you have reference-free variant calls (KisSNP2-format fasta / .snp file from ebwt2InDel) and an available reference genome, and you need reference-coordinate .vcf entries for downstream validation or comparison.

## When NOT to use

- No reference genome is available for the organism — ebwt2InDel is designed precisely for reference-free discovery, and alignment cannot place calls in reference coordinates
- Input reads contain characters other than A/C/G/T (e.g. 'N') — reads must be filtered or N replaced/removed before the upstream ebwt2InDel step
- You need genotypes/haplotypes rather than variant positions — this pipeline only maps mismatch/INDEL records to reference coordinates

## Inputs

- .snp file of SNP/INDEL calls produced by ebwt2InDel (KisSNP2-format fasta of variant contexts)
- Reference genome assembly in fasta format (hg38 Human assembly used in the article)

## Outputs

- Filtered calls file (output.5.snp) retaining variants covered by at least m reads
- FASTQ file of variant fragments with fake base qualities (calls.fastq)
- SAM alignment of variant fragments against the reference genome
- variants.vcf file with reference-coordinate variant entries

## How to apply

First filter the ebwt2InDel output with filter_snp keeping only variants supported by at least m reads (m=5 is suggested for datasets covered at >= 25x), producing output.5.snp. Convert the filtered calls to FASTQ with fake base qualities using seqtk (seqtk seq -F 'I'), then align these variant fragments against the hg38 Human reference assembly using bwa mem, yielding a SAM file. Translate the mismatches and INDEL records of the SAM alignment into VCF entries with sam2vcf; the article calls this chain seqtk -> bwa-mem -> sam2vcf the 'snp2vcf' pipeline. Finally remove duplicate .vcf entries, since analyzing both the dataset and its reverse-complement can report the same variant twice. The rationale is that ebwt2InDel is reference-free and outputs variant contexts, so alignment is only used post hoc to place calls in reference coordinates for VCF creation.

## Related tools

- **filter_snp** (Filters the ebwt2InDel .snp file to keep only SNP/INDELs covered by at least m reads before conversion) — github.com/nicolaprezza/ebwt2InDel
- **seqtk** (Converts the filtered call file to FASTQ with fake base qualities (seqtk seq -F 'I'))
- **bwa mem** (Aligns the variant-fragment FASTQ against the hg38 Human reference assembly, yielding a SAM file)
- **sam2vcf** (Translates the SAM's mismatch and INDEL records into VCF entries) — github.com/nicolaprezza/ebwt2InDel
- **hap.py** (Optionally validates the resulting .vcf calls against a gold-standard truth set (NCBI GIAB NA12878_HG001)) — Illumina/hap.py

## Examples

```
seqtk seq -F 'I' output.5.snp > calls.fastq && bwa mem hg38.fa calls.fastq > calls.sam && sam2vcf calls.sam > variants.vcf
```

## Evaluation signals

- variants.vcf exists and parses as a valid VCF
- VCF entries carry reference-coordinate positions consistent with the hg38 assembly
- Variant events present in the filtered calls.snp are represented in the output VCF (no missing events after conversion)
- Duplicate entries (same variant found on forward and reverse strands) have been removed from the .vcf
- Calls can be cross-validated against a ground-truth VCF with hap.py

## Limitations

- The .vcf conversion depends on an external reference genome, whereas ebwt2InDel itself is reference-free; alignment of short variant-context fragments may be ambiguous
- Duplicates appear in the .vcf because both the dataset and its reverse-complement are analyzed, so a deduplication step is required
- The paper notes real-data evaluation used an approximated ground truth, lowering precision estimates for the resulting calls

## Evidence

- [methods] snp2vcf pipeline composition: "We call **snp2vcf** the pipeline **seqtk -> bwa-mem -> sam2vcf**."
- [methods] filter_snp usage and threshold: "#Filter only events supported by at least 5 reads
filter_snp output.snp 5 > output.5.snp"
- [results] conversion to VCF against hg38: [evidence span withheld — anchor status: not_found]
- [results] deduplication of reverse-complement calls: [evidence span withheld — anchor status: not_found]
- [readme] reference-free scope of the upstream tool: "The **ebwt2InDel** suite can be used to discover SNPs/indels inside one or between two sets of reads (fasta/fastq) *without* aligning them to  a reference genome"
