---
name: paired-end-read-mapping-to-multi-genome-reference
description: Use when a metagenomic analysis needs paired-end shotgun reads mapped
  against many reference genomes simultaneously so junction read pairs spanning two
  different references can seed donor/receptor detection.
when_to_use_negative:
- Input is metatranscriptomic or long-read data rather than paired-end short-read
  shotgun NGS datasets
- The goal is variant calling or quantification on a single reference genome rather
  than identifying cross-reference junction reads for HGT breakpoint detection
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3697
tools:
- name: BWA
  role: Aligns paired-end shotgun reads against the combined multi-genome reference
    using 'bwa mem -M' to identify the set of donors and receptors via mapped junction
    reads
  repo: https://github.com/lichen2018/LEMON
- name: Samtools
  role: Converts the BWA SAM output to BAM and coordinate-sorts it ('samtools view
    -bhS' | 'samtools sort -o sample.bam') for downstream breakpoint detection
  repo: https://github.com/samtools/htslib
- name: LEMON
  role: Consumes the sorted BAM in get_raw_bkp.py to select junction reads mapping
    to two different references and detect raw HGT breakpoints
  repo: https://github.com/lichen2018/LEMON
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:a19ebb136c9f9beaeb5d8c311a9d1f2e431bbc540902309257cad391c4a08c25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - LEMON
  - Samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# paired-end-read-mapping-to-multi-genome-reference

## Summary

Aligns paired-end shotgun metagenomic reads against a multi-genome reference (all bacterial references indexed together) using BWA mem, retaining split/junction alignments, to identify candidate donor and receptor genomes for downstream HGT breakpoint detection in LEMON.

## When to use

Use when a metagenomic analysis needs paired-end shotgun reads mapped against many reference genomes simultaneously so junction read pairs spanning two different references can seed donor/receptor detection.

## When NOT to use

- Input is metatranscriptomic or long-read data rather than paired-end short-read shotgun NGS datasets
- The goal is variant calling or quantification on a single reference genome rather than identifying cross-reference junction reads for HGT breakpoint detection

## Inputs

- Metagenomic reference FASTA containing all reference genomes to be searched (Metagenomic_reference.fasta)
- Paired-end shotgun metagenomic reads as FASTQ (sample.1.fq, sample.2.fq)

## Outputs

- Sorted BAM file of mapped read pairs including junction pairs whose sides map to two different references (sample.bam)

## How to apply

First index the combined metagenomic reference FASTA (Metagenomic_reference.fasta) with BWA so all references are searched in one Burrows-Wheeler Transform index. Align the paired-end FASTQ files (sample.1.fq, sample.2.fq) with 'bwa mem -M -t 8' plus a read-group header (@RG ID/SM/LB); the -M flag keeps split alignments so junction pairs whose two sides map to two different references are retained rather than discarded. Convert the SAM output to BAM and coordinate-sort it with samtools ('samtools view -bhS' piped to 'samtools sort -o sample.bam'). The resulting sorted BAM is the input to LEMON's raw-breakpoint detection (get_raw_bkp.py), which selects junction reads mapping to different references to define the donor and receptor genome set.

## Related tools

- **BWA** (Aligns paired-end shotgun reads against the combined multi-genome reference using 'bwa mem -M' to identify the set of donors and receptors via mapped junction reads) — https://github.com/lichen2018/LEMON
- **Samtools** (Converts the BWA SAM output to BAM and coordinate-sorts it ('samtools view -bhS' | 'samtools sort -o sample.bam') for downstream breakpoint detection) — https://github.com/samtools/htslib
- **LEMON** (Consumes the sorted BAM in get_raw_bkp.py to select junction reads mapping to two different references and detect raw HGT breakpoints) — https://github.com/lichen2018/LEMON

## Examples

```
bwa mem -M -t 8 -R "@RG\tID:id\tSM:sample\tLB:lib" Metagenomic_reference.fasta sample.1.fq sample.2.fq | samtools view -bhS - | samtools sort -o sample.bam
```

## Evaluation signals

- sample.bam exists and is a valid, coordinate-sorted BAM (samtools view/quickcheck passes)
- BAM contains mapped read pairs, not just unmapped or singleton records
- Junction read pairs whose two sides map to two different references are present in the BAM, as required by get_raw_bkp.py
- Read-group header (@RG ID/SM/LB) is present in the BAM header
- Downstream get_raw_bkp.py runs without error on the unique-reads BAM

## Limitations

- Requires BWA 0.7.12+, HTSlib 1.9, and Samtools 1.3.1+ per the LEMON installation requirements
- Alignment is against a large combined reference (LEMON's reference set S: 16,093 species, 1,246,881 scaffolds), so indexing and mapping are computationally heavy; multi-threading (-t 8) is used to mitigate this
- -M flag must be retained; dropping it would lose split alignments needed to keep junction pairs with sides on different references

## Evidence

- [methods] Firstly, we map paired-end shotgun reads against reference genomes using Burrows Wheeler Aligner (BWA).: "Firstly, we map paired-end shotgun reads against reference genomes using Burrows Wheeler Aligner (BWA)."
- [methods] we utilize BWA[20] to align reads against the references to identify the set of donors and receptors: "we utilize BWA[20] to align reads against the references to identify the set of donors and receptors"
- [methods] Then, we select junction reads, whose two sides are mapped to two different references, from the set of mapped reads.: "Then, we select junction reads, whose two sides are mapped to two different references, from the set of mapped reads."
- [readme] bwa mem -M -t 8 -R "@RG\tID:id\tSM:sample\tLB:lib": "bwa mem -M -t 8 -R "@RG\tID:id\tSM:sample\tLB:lib""
