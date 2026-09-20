---
name: bwt-based-overlap-read-recruitment
description: Use when you have a SAM file of reads aligned to a partial or remotely
  related viral reference (e.g. bat coronavirus NC_014470.1) and need to recruit reads
  of the true target virus (e.g. SARS-CoV haplotypes) for strain assembly.
when_to_use_negative:
- The target virus shares no conservation with any available reference genome — 'A
  completely new virus that does not share any conservation with any reference genome
  won't be detected by our method.'
- Reads are uncorrected and exact overlap detection cannot be trusted (Karect correction
  is a required preprocessing step).
- You already have a complete, high-quality reference of the target virus, making
  direct read mapping more appropriate than overlap-based recruitment.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3697
tools:
- name: TAR-VIR
  role: Provides the build and overlap binaries that construct the read BWT/FM-index
    and recruit reads by overlap extension from the seed SAM file
  repo: https://github.com/chjiao/TAR-VIR
- name: Bowtie2
  role: Produces the seed-read SAM file (recommended default parameters with error
    function 'L,0,-0.6'; sweep of score-min values in the parameter study)
- name: BWA
  role: Alternative mapper for seed generation, sweeping mismatch penalty -B 1, 2,
    4, 8
- name: Karect
  role: Default error-correction tool; corrects substitutions/indels so exact overlap
    detection during recruitment is valid
  repo: https://github.com/aminallam/karect
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:95edd80272ba768e73e3f458c3a2427577cf85f3a6c66a62989fc5f669534e30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chjiao/TAR-VIR
  tools:
  - ART-illumina
  - BWA
  - Bowtie2
  - Karect
  - TAR-VIR
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# bwt-based-overlap-read-recruitment

## Summary

Recruit viral reads from a metagenomic dataset using the TAR-VIR overlap-extension module, which takes a seed-read SAM alignment and a BWT/FM-index of the reads to recruit additional reads sharing exact overlaps with seed reads. It targets viral strain reconstruction from remotely related or partial references.

## When to use

Use when you have a SAM file of reads aligned to a partial or remotely related viral reference (e.g. bat coronavirus NC_014470.1) and need to recruit reads of the true target virus (e.g. SARS-CoV haplotypes) for strain assembly.

## When NOT to use

- The target virus shares no conservation with any available reference genome — 'A completely new virus that does not share any conservation with any reference genome won't be detected by our method.'
- Reads are uncorrected and exact overlap detection cannot be trusted (Karect correction is a required preprocessing step).
- You already have a complete, high-quality reference of the target virus, making direct read mapping more appropriate than overlap-based recruitment.

## Inputs

- Error-corrected read set as FASTA (e.g. simulated MiSeq 250 bp paired-end reads from 5 SARS-CoV haplotypes plus Influenza NC_002023.1 and HCV NC_004102.1)
- SAM file of reads aligned to a partial or remotely related reference (e.g. Bowtie2 with score-min 'L,0,-0.6' against NC_014470.1, or BWA -B alignments)
- Overlap cutoff value (integer; > half read length, e.g. 150 for 250 bp reads)

## Outputs

- Fasta file of recruited reads (recruited_reads.fa) enriched for the target virus
- Recruited read counts per alignment parameter setting (e.g. 78,540 reads with BWA '-B 1')
- Per-haplotype genome coverage values (e.g. depths of 294/143/96/70/57x with 1.0 coverage for the five SARS-CoV haplotypes)

## How to apply

First correct sequencing errors with Karect, the pipeline's default corrector, since recruitment relies on exact overlap detection. Build the read index with the TAR-VIR 'build' module (./build -f reads.fa -o prefix), then run the 'overlap' module for each alignment SAM file produced by Bowtie2 (score-min 'L,0,-0.6' or sweeped values like -0.9) or BWA (mismatch penalty -B 1/2/4/8). Set the overlap cutoff longer than half the read size — for 250 bp MiSeq reads from ART-illumina (600 bp insert, 150 bp SD), use -c 150 — to prevent recruiting chimeric reads. Iterating recruitment against every Bowtie2 and BWA parameter setting lets you compare recruited read counts and per-haplotype genome coverage across seed-set configurations.

## Related tools

- **TAR-VIR** (Provides the build and overlap binaries that construct the read BWT/FM-index and recruit reads by overlap extension from the seed SAM file) — https://github.com/chjiao/TAR-VIR
- **Bowtie2** (Produces the seed-read SAM file (recommended default parameters with error function 'L,0,-0.6'; sweep of score-min values in the parameter study))
- **BWA** (Alternative mapper for seed generation, sweeping mismatch penalty -B 1, 2, 4, 8)
- **Karect** (Default error-correction tool; corrects substitutions/indels so exact overlap detection during recruitment is valid) — https://github.com/aminallam/karect

## Examples

```
./build -f test_data/virus.fa -o virus; ./overlap -S test_data/HIV.sam -x virus -f test_data/virus.fa -c 180 -o virus_recruit.fa
```

## Evaluation signals

- On the README test set (virus.fa + HIV.sam with 287 aligned seed reads), overlap recruitment with -c 180 must yield 8008 recruited reads.
- Reproducing the SARS-CoV sweep, BWA '-B 1' seed set should yield 78,540 recruited reads with per-haplotype depths 294/143/96/70/57x and genome coverage 1.0 for all five haplotypes.
- Bowtie2 'L,0,-0.9' should recruit 52,377 reads covering 20%-30% more of the five haplotype genomes than the 52,688 reads BWA -B 1 aligns.
- Check recruited reads for contamination: all recruited reads should originate from SARS-CoV with none from Influenza or HCV.

## Limitations

- A completely new virus sharing no conservation with any reference genome will not be detected by the recruitment stage.
- Recruitment depends on prior read error correction; exact overlap matching fails on raw, error-containing reads.
- No software version numbers are provided for TAR-VIR, Bowtie2, or BWA, and no specific recommended overlap threshold value is stated beyond 'longer than half of the read size'.

## Evidence

- [readme] ./overlap -S align.sam -x prefix -f reads.fa -c overlap_cutoff -o recruited_reads.fa: "./overlap -S align.sam -x prefix -f reads.fa -c overlap_cutoff -o recruited_reads.fa"
- [methods] we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads: "we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads"
- [results] BWA can align slightly more reads than what Bowtie2 can recruit with the parameter “L, 0, -0.9” ((52,688 vs. 52,377).: [evidence span withheld — anchor status: not_found]
- [readme] If everything is good, the recruited reads number should be 8008.: "With these aligned (287) reads, more HIV-1 reads can be recruited from the viral metagenomics data set."
