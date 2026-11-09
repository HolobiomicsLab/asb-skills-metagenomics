---
name: per-haplotype-read-coverage-quantification
description: Use when comparing read-recruitment strategies for viral metagenomic
  data and you need recruited-read counts plus per-haplotype genome coverage/fraction
  to judge which mapper parameters enrich the target virus .
when_to_use_negative:
- The target virus shares no conservation with any available reference genome — TAR-VIR's
  recruitment cannot detect it
- You only have raw mapper alignment counts and need no coverage-based comparison
  — alignment counts alone (52,688 vs 52,377) can mislead since recruited reads may
  cover 20%-30% more genome
- Reads have not been error-corrected — exact overlap detection will fail on error-containing
  reads
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3697
tools:
- name: TAR-VIR
  role: Builds read FM-index ('build') and recruits target viral reads via overlap
    extension ('overlap' with -c overlap cutoff) from mapper SAM files
  repo: https://github.com/chjiao/TAR-VIR
- name: Bowtie2
  role: Aligns reads against the related reference using score-min functions such
    as 'L,0,-0.6', '-0.35', '-0.9', '-1.2' to generate seed-read SAM files
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: BWA
  role: Alternative aligner with mismatch penalty sweep -B 1, 2, 4, 8 producing comparison
    SAM files
- name: Karect
  role: Corrects substitution/indel sequencing errors so TAR-VIR's exact overlap detection
    is valid
  repo: https://github.com/aminallam/karect
- name: ART-illumina
  role: Simulates error-containing MiSeq paired-end reads (250 bp, 600 bp insert,
    150 bp SD) for benchmark coverage quantification
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:f81b29f13d69f5a3e5d039910459b8029ac9d181d2be6ff73ff98804c3d9731c
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
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Per-haplotype read coverage quantification

## Summary

Quantifies how many reads TAR-VIR's overlap-recruitment stage pulls in and how deeply and completely each viral haplotype genome is covered, enabling comparison of read-mapper seed parameter sweeps (e.g. Bowtie2 score-min 'L,0,-x' vs BWA -B) for targeted viral read enrichment.

## When to use

Use when comparing read-recruitment strategies for viral metagenomic data and you need recruited-read counts plus per-haplotype genome coverage/fraction to judge which mapper parameters enrich the target virus best.

## When NOT to use

- The target virus shares no conservation with any available reference genome — TAR-VIR's recruitment cannot detect it
- You only have raw mapper alignment counts and need no coverage-based comparison — alignment counts alone (52,688 vs 52,377) can mislead since recruited reads may cover 20%-30% more genome
- Reads have not been error-corrected — exact overlap detection will fail on error-containing reads

## Inputs

- Error-corrected paired-end viral metagenomic read set (e.g. ART-illumina MiSeq simulation: 250 bp reads, 600 bp mean insert, 150 bp SD)
- SAM alignment files of the reads against a related reference genome (one per Bowtie2 score-min or BWA -B setting)
- TAR-VIR read index built via './build -f reads.fa -o prefix'
- Known target haplotype reference genomes for coverage calculation (e.g. 5 SARS-CoV haplotypes from NC_004718.3)

## Outputs

- Recruited reads FASTA per alignment setting (recruited_reads.fa)
- Recruited read count per mapper/parameter setting
- Per-haplotype sequencing depth (e.g. 294/143/96/70/57x)
- Per-haplotype genome coverage fraction (e.g. 1.0 for all five haplotypes)
- Comparison table across Bowtie2 and BWA parameter sweeps

## How to apply

First simulate or obtain error-containing paired-end reads, correct them with Karect (TAR-VIR's default error-correction step, required so exact overlap detection works), and align them to a remotely related reference (e.g. bat coronavirus NC_014470.1 for SARS-CoV haplotypes) with each mapper setting to be compared. Build the read FM-index with TAR-VIR 'build' and run the 'overlap' module for every alignment SAM file using an overlap cutoff greater than half the 250 bp read length (e.g. 150). For each run, count recruited reads and compute per-haplotype sequencing depth and genome-coverage fraction by mapping recruited reads back to each known haplotype. Judge success by reproducing the expected pattern: e.g. BWA '-B 1' aligns 52,688 reads but Bowtie2 'L,0,-0.9' recruits 52,377 reads covering 20%-30% more of the five haplotype genomes, with the '-B 1' seed set reaching 78,540 reads at 294/143/96/70/57x depth and 1.0 coverage for all five SARS-CoV haplotypes.

## Related tools

- **TAR-VIR** (Builds read FM-index ('build') and recruits target viral reads via overlap extension ('overlap' with -c overlap cutoff) from mapper SAM files) — https://github.com/chjiao/TAR-VIR
- **Bowtie2** (Aligns reads against the related reference using score-min functions such as 'L,0,-0.6', '-0.35', '-0.9', '-1.2' to generate seed-read SAM files) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **BWA** (Alternative aligner with mismatch penalty sweep -B 1, 2, 4, 8 producing comparison SAM files)
- **Karect** (Corrects substitution/indel sequencing errors so TAR-VIR's exact overlap detection is valid) — https://github.com/aminallam/karect
- **ART-illumina** (Simulates error-containing MiSeq paired-end reads (250 bp, 600 bp insert, 150 bp SD) for benchmark coverage quantification)

## Examples

```
./build -f reads.fa -o reads_idx; ./overlap -S align.sam -x reads_idx -f reads.fa -c 150 -o recruited_reads.fa
```

## Evaluation signals

- Recruited-read counts per setting: BWA '-B 1' aligns 52,688 reads; Bowtie2 'L,0,-0.9' recruits 52,377 reads; the '-B 1' seed set ultimately recruits 78,540 reads
- Per-haplotype depths reach 294/143/96/70/57x with genome coverage 1.0 for all five SARS-CoV haplotypes under the BWA '-B 1' seed set
- No contamination: all recruited reads originate from the target virus (no Influenza NC_002023.1 or HCV NC_004102.1 reads)
- Overlap cutoff must exceed half the read length (150 for 250 bp reads) to prevent recruiting chimeric reads
- Coverage-based comparison: Bowtie2 'L,0,-0.9' recruited reads cover 20%-30% more haplotype genome than BWA -B 1 aligned reads despite fewer counts

## Limitations

- No software version numbers are provided for TAR-VIR, Bowtie2, BWA, or PEHaplo, so exact reproducibility of counts/coverage may vary across versions
- A completely new virus sharing no conservation with any reference genome will not be detected by this recruitment stage
- When using TAR-VIR in local alignment mode, no quantitative criteria are given for when alignment length and score should accept or reject reads
- Recruited reads usually mix single- and paired-end reads, requiring PEHaplo's '-f' single-fasta input option for downstream assembly

## Evidence

- [results] BWA can align slightly more reads than what Bowtie2 can recruit with the parameter “L, 0, -0.9” ((52,688 vs. 52,377). The recruited reads (52,377), however, cover 20%-30% more genomes: [evidence span withheld — anchor status: not_found]
- [results] Worth noting is all the recruited reads are from SARS-CoV (no contamination from Influenza and HBV).: "Worth noting is all the recruited reads are from SARS-CoV (no contamination from Influenza and HBV)."
- [methods] we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads: "we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads"
- [readme] ./overlap -S test_data/HIV.sam -x virus -f test_data/virus.fa -c 180 -o virus_recruit.fa: "./overlap -S test_data/HIV.sam -x virus -f test_data/virus.fa -c 180 -o virus_recruit.fa"
- [results] In total, there are 173,703 simulated reads, of which 119,002 reads are from the five SARS-CoV haplotypes.: "In total, there are 173,703 simulated reads, of which 119,002 reads are from the five SARS-CoV haplotypes."
