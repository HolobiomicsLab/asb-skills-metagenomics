---
name: adapter-trimming-quality-control
description: Use when raw paired-end Illumina metagenomics reads (e.g. FASTQ/FASTQ.gz
  such as SRA SRR5381736) must be adapter-trimmed and quality-filtered prior to 16S/18S
  read extraction with SortMeRNA or BBMap.
when_to_use_negative:
- Input is already trimmed FASTQ or a filtered 16S/18S read set from SortMeRNA — re-trimming
  would risk over-trimming before SSU extraction
- Data are long-read (e.g. PacBio/Oxford Nanopore) rather than Illumina FASTQ, since
  the BBTools parameters here (ktrim, kmer=21, qtrim=rl) target short Illumina reads
- The analysis goal is classification of already-reconstructed full-length SSU sequences
  (handled by QIIME 2 sklearn classifier/VSEARCH), not read preprocessing
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0196
tools:
- name: BBTools
  role: Adapter removal and read trimming of Illumina metagenomic FASTQ inputs prior
    to SSU read extraction
  repo: https://github.com/oschakoory/RiboTaxa
- name: FastQC
  role: Per-file quality control of raw and trimmed FASTQ reads within the RiboTaxa
    pipeline
  repo: https://github.com/oschakoory/RiboTaxa
- name: MultiQC
  role: Aggregation of FastQC quality-control reports across samples
  repo: https://github.com/MultiQC/MultiQC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:28d5cb84e7c95cfa09ffcecad93a855febc6589a9549e3b642b0bc03954a35a8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  repo_url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBTools
  - FastQC
  - MultiQC
  - SortMeRNA
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# adapter-trimming-quality-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess Illumina (meta)genomic FASTQ reads with FastQC/MultiQC quality control and BBTools adapter/quality trimming (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1) before SSU read extraction and rRNA gene reconstruction, as done in the RiboTaxa pipeline.

## When to use

Use when raw paired-end Illumina metagenomics reads (e.g. FASTQ/FASTQ.gz such as SRA SRR5381736) must be adapter-trimmed and quality-filtered prior to 16S/18S read extraction with SortMeRNA or BBMap.

## When NOT to use

- Input is already trimmed FASTQ or a filtered 16S/18S read set from SortMeRNA — re-trimming would risk over-trimming before SSU extraction
- Data are long-read (e.g. PacBio/Oxford Nanopore) rather than Illumina FASTQ, since the BBTools parameters here (ktrim, kmer=21, qtrim=rl) target short Illumina reads
- The analysis goal is classification of already-reconstructed full-length SSU sequences (handled by QIIME 2 sklearn classifier/VSEARCH), not read preprocessing

## Inputs

- Raw paired-end or single-end Illumina FASTQ reads (compressed .fastq.gz or uncompressed .fastq), e.g. SRA SRR5381736 mock community reads

## Outputs

- Adapter- and quality-trimmed FASTQ read files (R1/R2) meeting minlength=60, trimq=20, maxns=1 criteria
- FastQC/MultiQC quality reports on raw reads

## How to apply

Run FastQC (and optionally MultiQC) on the raw single-end or paired-end FASTQ files to check per-base quality before trimming. Then trim with BBTools: remove adapter-matching bases with ktrim=r using kmer=21, apply quality trimming with trimq=20 and qtrim=rl (trimming both ends), discard reads shorter than minlength=60 bases after trimming, and replace/discard reads with maxns=1 ambiguous bases. The cleaned FASTQ output is the input for subsequent 16S/18S filtering (SortMeRNA/BBMap against indexed SILVA SSU 138.1) and full-length SSU rRNA reconstruction with EMIRGE (Emirge.py/Emirge_amplicon.py), MATAM and MetaRib. Correct application is judged by the trimmed reads passing QC and by downstream SSU reconstruction yielding full-length rRNA sequences.

## Related tools

- **BBTools** (Adapter removal and read trimming of Illumina metagenomic FASTQ inputs prior to SSU read extraction) — https://github.com/oschakoory/RiboTaxa
- **FastQC** (Per-file quality control of raw and trimmed FASTQ reads within the RiboTaxa pipeline) — https://github.com/oschakoory/RiboTaxa
- **MultiQC** (Aggregation of FastQC quality-control reports across samples) — https://github.com/MultiQC/MultiQC

## Examples

```
bbduk.sh in=R1.fastq.gz in2=R2.fastq.gz out=R1.trimmed.fq out2=R2.trimmed.fq ktrim=r kmer=21 minlength=60 trimq=20 qtrim=rl maxns=1
```

## Evaluation signals

- Trimmed FASTQ files no longer contain adapter-matching k-mers (ktrim=r, kmer=21) as verifiable by re-running FastQC adapter-content plots
- No retained read is shorter than minlength=60 bases, and reads with more than maxns=1 ambiguous bases are discarded
- FastQC/MultiQC reports show improved per-base quality after trimq=20/qtrim=rl trimming
- Downstream SortMeRNA/BBMap 16S extraction and EMIRGE/MetaRib reconstruction of the trimmed reads reproduces full-length SSU sequences from the mock community (41 expected 16S rRNA variants, 20 detectable microorganisms via the Emirge_amplicon.py + MetaRib combination)
- All samples in a batch have trimming statistics reported (e.g. via MultiQC aggregation) so no sample silently failed preprocessing

## Limitations

- Trimmings are calibrated for Illumina short reads; RiboTaxa's trimming parameters are not validated for other platforms
- Aggressive trimming (minlength=60, maxns=1) may discard marginal reads, reducing the SSU read pool available for reconstruction
- RiboTaxa's trimming quality depends on correctly indexed external databases downstream (SILVA SSU 138.1); trimming alone does not guarantee species-level assignment

## Evidence

- [methods] BBTools trimming parameters: [evidence span withheld — anchor status: not_found]
- [discussion] RiboTaxa scope of read trimming: "RiboTaxa is a user-friendly metagenomic classifier which supports database preparation, read trimming and SSU read extraction, SSU rRNA gene targeted assembly and taxonomic profiling."
- [readme] BBTools role in pipeline: "For adapters removal and trimming: BBTOOLS"
- [readme] FastQC/MultiQC role in pipeline: [evidence span withheld — anchor status: not_found]
- [readme] Input format support: "RiboTaxa takes as input singled-end or paired-end files which can be in compressed format (fastq.gz) or uncompressed format (.fastq)."
