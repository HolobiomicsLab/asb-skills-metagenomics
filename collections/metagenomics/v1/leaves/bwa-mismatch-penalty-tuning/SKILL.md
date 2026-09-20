---
name: bwa-mismatch-penalty-tuning
description: Use when reads must be recruited to a target virus via a distant reference
  genome and you need to sweep BWA mismatch penalties (e.g., -B 1/2/4/8) to build
  seed-read sets for overlap-based recruitment.
when_to_use_negative:
- A complete, high-quality reference genome of the target virus is available — plain
  read mapping suffices and overlap extension is unnecessary
- The target virus shares no conservation with any reference genome — TAR-VIR cannot
  detect it
- Reads are raw and uncorrected — exact overlap detection requires prior error correction
  (Karect), so penalty tuning would propagate mapping errors
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: BWA
  role: Aligns reads against the remote reference under varying mismatch penalties
    (-B 1/2/4/8) to generate candidate seed-read SAM files
- name: TAR-VIR
  role: Build module creates the read FM-index; overlap module recruits additional
    target reads from each SAM using an overlap cutoff of 150
  repo: https://github.com/chjiao/TAR-VIR
- name: Karect
  role: Corrects substitution/indel errors in reads so exact overlap detection works
    downstream
- name: Bowtie2
  role: Comparator aligner swept over score-min functions ('L,0,-0.6', -0.35, -0.9,
    -1.2) to benchmark seed sets against BWA -B settings
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:a230ae71e75e9df3f95b71c237f8a0ef8919323548b396097a8cbf2836294846
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
n_claims: 14
---

# bwa-mismatch-penalty-tuning

## Summary

Tune BWA's mismatch penalty (-B 1, 2, 4, 8) when aligning metagenomic reads against a remotely related viral reference, then feed each SAM into TAR-VIR's overlap recruitment stage to find the seed set that maximizes recruited-read genome coverage rather than raw alignment counts.

## When to use

Use when reads must be recruited to a target virus via a distant reference genome and you need to sweep BWA mismatch penalties (e.g., -B 1/2/4/8) to build seed-read sets for overlap-based recruitment.

## When NOT to use

- A complete, high-quality reference genome of the target virus is available — plain read mapping suffices and overlap extension is unnecessary
- The target virus shares no conservation with any reference genome — TAR-VIR cannot detect it
- Reads are raw and uncorrected — exact overlap detection requires prior error correction (Karect), so penalty tuning would propagate mapping errors

## Inputs

- Error-corrected paired-end metagenomic reads (e.g., ART-illumina-simulated MiSeq 250 bp reads) in FASTA
- Remotely related reference genome (e.g., NC_014470.1 bat coronavirus) in FASTA
- BWA alignment SAM files produced under mismatch penalty settings -B 1, 2, 4, and 8
- TAR-VIR read index prefix (built from reads.fa)

## Outputs

- Per-setting recruited-read FASTA files from TAR-VIR overlap (-c 150)
- Table of recruited read counts per BWA -B setting (e.g., 78,540 reads for -B 1)
- Per-haplotype coverage depths (e.g., 294/143/96/70/57x) and genome coverage fractions for the five SARS-CoV haplotypes
- Contamination check of recruited reads against off-target genomes (Influenza, HCV)

## How to apply

Align the (error-corrected, e.g., Karect) metagenomic reads against the partially related reference (e.g., bat coronavirus NC_014470.1 for SARS-CoV targets) with BWA several times, varying the mismatch penalty via -B 1, 2, 4, and 8, keeping each SAM output. Build the read FM-index with TAR-VIR's 'build' module (./build -f reads.fa -o prefix), then run the 'overlap' module per alignment with an overlap cutoff of 150 (greater than half the 250 bp read length, as TAR-VIR sets the threshold longer than half the read size to avoid recruiting chimeric reads). Evaluate each seed set by recruited read counts and per-haplotype genome coverage (depth and fraction of the five SARS-CoV haplotype genomes covered), not by the number of reads BWA aligned. The rationale is that a stricter or looser mismatch penalty changes which reads become seeds, and the overlap extension stage, not alignment count, determines final sensitivity.

## Related tools

- **BWA** (Aligns reads against the remote reference under varying mismatch penalties (-B 1/2/4/8) to generate candidate seed-read SAM files)
- **TAR-VIR** (Build module creates the read FM-index; overlap module recruits additional target reads from each SAM using an overlap cutoff of 150) — https://github.com/chjiao/TAR-VIR
- **Karect** (Corrects substitution/indel errors in reads so exact overlap detection works downstream)
- **Bowtie2** (Comparator aligner swept over score-min functions ('L,0,-0.6', -0.35, -0.9, -1.2) to benchmark seed sets against BWA -B settings)

## Examples

```
./build -f reads.fa -o reads && ./overlap -S bwa_B1.sam -x reads -f reads.fa -c 150 -o recruited_reads.fa
```

## Evaluation signals

- Recruited-read count matches reported values, e.g., 78,540 reads recruited with the BWA -B 1 seed set
- Per-haplotype depths of 294/143/96/70/57x with genome coverage of 1.0 for all five SARS-CoV haplotypes
- BWA -B 1 aligns 52,688 reads vs 52,377 recruited by Bowtie2 'L, 0, -0.9', yet recruited reads cover 20%-30% more of the haplotype genomes — coverage, not alignment count, is the metric
- All recruited reads originate from SARS-CoV with no contamination from Influenza or HCV
- Overlap cutoff is set above half the read length (150 for 250 bp reads) to exclude chimeric reads

## Limitations

- A completely new virus sharing no conservation with any reference genome will not be detected by this seed-based approach
- In local alignment mode, users must screen mapping results using alignment length and score; no quantitative accept/reject criteria are given
- The paper provides no software version numbers for BWA/Bowtie2/TAR-VIR, so exact parameter behavior may vary across releases

## Evidence

- [results] BWA can align slightly more reads than what Bowtie2 can recruit with the parameter “L, 0, -0.9” ((52,688 vs. 52,377).: [evidence span withheld — anchor status: not_found]
- [methods] we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads: "we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads"
- [results] Worth noting is all the recruited reads are from SARS-CoV (no contamination from Influenza and HBV).: "Worth noting is all the recruited reads are from SARS-CoV (no contamination from Influenza and HBV)."
- [readme] You need to conduct error correction for the reads. By dafault, we use karect: "You need to conduct error correction for the reads. By dafault, we use karect"
- [readme] The recruited reads usually contain both single- or paired-end reads, use the '-f' option of PEHaplo to input one fasta file.: "The recruited reads usually contain both single- or paired-end reads, use the '-f' option of PEHaplo to input one fasta file."
