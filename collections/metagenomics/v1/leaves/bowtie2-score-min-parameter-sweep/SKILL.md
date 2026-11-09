---
name: bowtie2-score-min-parameter-sweep
description: Use when aligning metagenomic reads against a partial or remotely related
  viral reference (e.g. bat coronavirus NC_014470.1) and you need to choose the read-mapping
  scoring that yields the richest seed set for overlap-based recruitment.
when_to_use_negative:
- The reference genome is complete and high-quality, so straightforward read mapping
  suffices without a parameter sweep
- A completely new virus shares no conservation with any reference genome — TAR-VIR's
  mapping-based recruiting (and hence this sweep) cannot detect it
- Reads have not been error-corrected (e.g. with Karect), since TAR-VIR's overlap
  recruitment relies on exact overlaps
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: Bowtie2
  role: Primary read mapper whose --score-min function is swept to generate candidate
    seed alignments (recommended baseline 'L,0,-0.6')
- name: BWA
  role: Comparison mapper with mismatch-penalty sweep (-B 1/2/4/8) to benchmark Bowtie2
    seed quality
- name: TAR-VIR
  role: Overlap-extension module ('build' + 'overlap') that recruits reads from each
    SAM seed alignment using the overlap cutoff
  repo: https://github.com/chjiao/TAR-VIR
- name: Karect
  role: Default error-correction tool applied before alignment so exact overlap detection
    works
  repo: https://github.com/aminallam/karect
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:42c1964281615ba24630a2651492639bad9d35dc3e6e97e50cfac9c434451774
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
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# bowtie2-score-min-parameter-sweep

## Summary

Sweeps Bowtie2 score-min settings (e.g. 'L,0,-0.6' through 'L,0,-1.2') against BWA mismatch-penalty (-B) settings to find the seed-read alignment that maximizes downstream TAR-VIR overlap recruitment of viral reads.

## When to use

Use when aligning metagenomic reads against a partial or remotely related viral reference (e.g. bat coronavirus NC_014470.1) and you need to choose the read-mapping scoring that yields the richest seed set for overlap-based recruitment.

## When NOT to use

- The reference genome is complete and high-quality, so straightforward read mapping suffices without a parameter sweep
- A completely new virus shares no conservation with any reference genome — TAR-VIR's mapping-based recruiting (and hence this sweep) cannot detect it
- Reads have not been error-corrected (e.g. with Karect), since TAR-VIR's overlap recruitment relies on exact overlaps

## Inputs

- Karect error-corrected paired-end metagenomic reads (e.g. 173,703 simulated MiSeq reads, 250 bp, insert 600 bp SD 150)
- A partial or remotely related reference genome (e.g. bat coronavirus NC_014470.1) as a Bowtie2 index
- Bowtie2 score-min settings to sweep: 'L,0,-0.6', 'L,0,-0.35', 'L,0,-0.9', 'L,0,-1.2'
- BWA -B mismatch-penalty settings for the comparison sweep: 1, 2, 4, 8

## Outputs

- One SAM alignment file per Bowtie2 score-min setting (and per BWA -B setting)
- Per-setting recruited-read FASTA files from the TAR-VIR overlap module
- A table of recruited read counts and per-haplotype genome coverage/depth for each parameter setting

## How to apply

Align the (Karect-corrected) read set against the reference with Bowtie2 in several runs, varying the --score-min function: baseline 'L,0,-0.6' (the TAR-VIR recommendation), then looser 'L,0,-0.35', 'L,0,-0.9', and 'L,0,-1.2', keeping each output as a SAM file. Optionally repeat with BWA mismatch penalties -B 1, 2, 4, and 8 for comparison. For each SAM file, build the read FM-index with TAR-VIR 'build' and run the 'overlap' module with an overlap cutoff of 150 (longer than half the 250 bp read length, to avoid recruiting chimeric reads). Compare recruited read counts and per-haplotype genome coverage across settings; looser score-min functions such as 'L,0,-0.9' can recruit fewer total reads than BWA (52,377 vs 52,688) yet cover 20%-30% more of each haplotype's genome, so coverage — not raw alignment count — drives the choice.

## Related tools

- **Bowtie2** (Primary read mapper whose --score-min function is swept to generate candidate seed alignments (recommended baseline 'L,0,-0.6'))
- **BWA** (Comparison mapper with mismatch-penalty sweep (-B 1/2/4/8) to benchmark Bowtie2 seed quality)
- **TAR-VIR** (Overlap-extension module ('build' + 'overlap') that recruits reads from each SAM seed alignment using the overlap cutoff) — https://github.com/chjiao/TAR-VIR
- **Karect** (Default error-correction tool applied before alignment so exact overlap detection works) — https://github.com/aminallam/karect

## Examples

```
bowtie2 -x NC_014470.1 --score-min 'L,0,-0.9' -1 reads_1.fa -2 reads_2.fa -S align_L0_0.9.sam; ./overlap -S align_L0_0.9.sam -x virus -f reads.fa -c 150 -o recruited_reads.fa
```

## Evaluation signals

- With score-min 'L,0,-0.9', TAR-VIR recruits ~52,377 reads — fewer than BWA -B 1's 52,688 aligned reads, but covering 20%-30% more of each of the five SARS-CoV haplotype genomes
- With the BWA '-B 1' seed set, TAR-VIR recruits 78,540 reads with per-haplotype depths of 294/143/96/70/57x and genome coverage of 1.0 for all five haplotypes
- All recruited reads derive from SARS-CoV — no contamination from Influenza (NC_002023.1) or HCV/HBV
- The overlap cutoff is set above half the read length (e.g. 150 for 250 bp reads) to prevent recruiting chimeric reads
- Each alignment is retained as a separate SAM file so recruited counts can be tabulated per score-min / -B setting

## Limitations

- The paper provides no software version numbers for Bowtie2, BWA, or TAR-VIR, so exact reproducibility of the sweep may vary across releases
- No quantitative criteria are given for accepting/rejecting reads when local-mode screening via alignment length and score is used
- A completely new virus sharing no conservation with any reference genome will not be detected by this mapping-dependent approach

## Evidence

- [results] BWA can align slightly more reads than what Bowtie2 can recruit with the parameter “L, 0, -0.9” ((52,688 vs. 52,377).: [evidence span withheld — anchor status: not_found]
- [methods] we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads: "we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads"
- [readme] We recommend to use Bowtie2 with default parameters and the allowed error function "L,0,-0.6".: "We recommend to use Bowtie2 with default parameters and the allowed error function "L,0,-0.6"."
- [readme] ./overlap -S align.sam -x prefix -f reads.fa -c overlap_cutoff -o recruited_reads.fa: "./overlap -S align.sam -x prefix -f reads.fa -c overlap_cutoff -o recruited_reads.fa"
