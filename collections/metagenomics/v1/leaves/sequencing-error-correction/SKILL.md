---
name: sequencing-error-correction
description: Use when exact-overlap-based read recruitment (e.g., TAR-VIR overlap
  extension with an overlap cutoff) is planned downstream, since raw sequencing errors
  would break exact overlap matching between reads.
when_to_use_negative:
- The downstream pipeline tolerates mismatches (e.g., seed-based mapping alone with
  Bowtie2/BWA) and does not rely on exact read overlaps
- Reads are already processed through a correction step; re-correcting can degrade
  assembly quality (PEHaplo expects raw corrected inputs, not double-corrected data)
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: Karect
  role: Default error-correction tool applied to raw reads before TAR-VIR overlap
    recruitment, correcting substitution, insertion, and deletion errors
  repo: https://github.com/aminallam/karect
- name: TAR-VIR
  role: Downstream pipeline whose overlap extension module ('build' + 'overlap') depends
    on corrected reads for exact overlap detection
  repo: https://github.com/chjiao/TAR-VIR
- name: PEHaplo
  role: Haplotype assembly tool that consumes recruited reads after correction and
    internally invokes karect/sga via its -t threads option
  repo: https://github.com/chjiao/PEHaplo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:b799b31de6ad388f8f4f044e85a12d503c0d538990e04267d034646765c2db9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aminallam/karect
  tools:
  - ART-illumina
  - BWA
  - Karect
  - PEHaplo
  - TAR-VIR
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# sequencing-error-correction

## Summary

Correct substitution, insertion, and deletion errors in NGS metagenomic reads (e.g., simulated MiSeq 250 bp paired-end reads) with Karect before overlap-based viral read recruitment, so that exact read-overlap detection in TAR-VIR's overlap extension module works reliably.

## When to use

Use when exact-overlap-based read recruitment (e.g., TAR-VIR overlap extension with an overlap cutoff) is planned downstream, since raw sequencing errors would break exact overlap matching between reads.

## When NOT to use

- The downstream pipeline tolerates mismatches (e.g., seed-based mapping alone with Bowtie2/BWA) and does not rely on exact read overlaps
- Reads are already processed through a correction step; re-correcting can degrade assembly quality (PEHaplo expects raw corrected inputs, not double-corrected data)

## Inputs

- Raw metagenomic paired-end read set (e.g., 173,703 simulated MiSeq reads, 250 bp, FASTA/FASTQ: reads.fa)
- Alignment SAM file of reads against a related reference (e.g., bat coronavirus NC_014470.1) used downstream by the overlap module

## Outputs

- Error-corrected read set (corrected reads.fa) suitable for exact-overlap detection and Bowtie2/BWA alignment
- Recruited reads FASTA (recruited_reads.fa) produced by the TAR-VIR overlap module from corrected reads

## How to apply

Run Karect (the TAR-VIR pipeline's default error-correction tool) on the raw FASTQ/FASTA read set — for the SARS-CoV benchmark this is 173,703 error-containing ART-illumina MiSeq paired-end reads (250 bp, 600 bp mean insert, 150 bp SD) simulated from 5 SARS-CoV haplotypes plus Influenza and HCV genomes — before any alignment or overlap extension. Karect corrects substitution, insertion, and deletion errors so that reads sharing true sequence can be found by the TAR-VIR 'overlap' module using exact overlaps. After correction, build the read FM/BWT index with './build' and recruit reads with './overlap' using a cutoff set greater than half the read length (e.g., 150 for 250 bp reads; 180 in the HIV test example). Error correction should precede both the mapper alignment (Bowtie2/BWA) and the overlap recruitment so seed-read alignments and exact overlaps are computed on corrected sequences.

## Related tools

- **Karect** (Default error-correction tool applied to raw reads before TAR-VIR overlap recruitment, correcting substitution, insertion, and deletion errors) — https://github.com/aminallam/karect
- **TAR-VIR** (Downstream pipeline whose overlap extension module ('build' + 'overlap') depends on corrected reads for exact overlap detection) — https://github.com/chjiao/TAR-VIR
- **PEHaplo** (Haplotype assembly tool that consumes recruited reads after correction and internally invokes karect/sga via its -t threads option) — https://github.com/chjiao/PEHaplo

## Examples

```
./build -f test_data/virus.fa -o virus ; ./overlap -S test_data/HIV.sam -x virus -f test_data/virus.fa -c 180 -o virus_recruit.fa
```

## Evaluation signals

- With the HIV test data (virus.fa, HIV.sam) and overlap cutoff 180, the recruited read count must be 8008 — confirming correction and index building were done correctly
- For the SARS-CoV sweep, using BWA '-B 1' seed set, TAR-VIR should recruit 78,540 reads with per-haplotype depths of 294/143/96/70/57x and genome coverage 1.0 for all five haplotypes
- All recruited reads should derive from SARS-CoV with no contamination from Influenza or HCV/HBV reads
- Recruited reads should cover 20%-30% more of the five haplotype genomes than the aligned seed set (52,377 recruited vs. 52,688 BWA-aligned reads)

## Limitations

- Karect correction must be applied before overlap extension; skipping it prevents exact overlap detection from working
- Correction does not help reads from a virus sharing no conservation with any reference genome, since seeds cannot be obtained
- No software version numbers are given for Karect, TAR-VIR, Bowtie2, or BWA in the paper or README
- PEHaplo pipeline requires Python 2.7.x and networkx 1.11, which constrains the runtime environment for the post-correction assembly

## Evidence

- [readme] Default error correction before overlap recruitment: "You need to conduct error correction for the reads. By dafault, we use karect"
- [results] Corrected reads enable exact overlap recruitment: [evidence span withheld — anchor status: not_found]
- [methods] Overlap cutoff rationale relative to read length: "we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads"
- [results] Expected recruitment numbers validating correction and overlap settings: [evidence span withheld — anchor status: not_found]
- [readme] README test-case expected output count: "If everything is good, the recruited reads number should be 8008."
