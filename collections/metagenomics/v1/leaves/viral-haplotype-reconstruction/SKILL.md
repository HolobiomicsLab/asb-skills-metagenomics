---
name: viral-haplotype-reconstruction
description: Use when a viral metagenomic read set (e.g. ART-illumina MiSeq 250 bp
  paired-end reads) must be classified against a partial or remotely related reference
  and resolved into strain-level haplotype contigs without a high-quality reference
  genome.
when_to_use_negative:
- The target virus shares no conservation with any reference genome — TAR-VIR's read
  recruiting will not detect a completely new virus
- Only generic species-level abundance profiling is needed and strain-level haplotypes
  are not of interest (generic assemblers suffice)
- A complete high-quality reference genome exists for the exact strain, making reference-based
  haplotype reconstruction tools more appropriate
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: TAR-VIR
  role: overlap-extension module that recruits target-viral reads from the SAM seed
    alignments into recruited_reads.fa using an overlap cutoff
  repo: https://github.com/chjiao/TAR-VIR
- name: PEHaplo
  role: de novo haplotype reconstruction from recruited reads via paired-end overlap
    graph and greedy path search; accepts a single mixed-read fasta via '-f'
  repo: github.com/chjiao/PEHaplo
- name: Bowtie2
  role: aligns reads to the partial reference to generate seed-read SAM input; recommended
    error function 'L,0,-0.6' (sweep included 'L,0,-0.9')
- name: SPAdes
  role: baseline generic de novo assembler run on the identical recruited read set
    for comparison
- name: SGA
  role: baseline string-graph assembler used by PEHaplo internally and as an external
    comparison assembler
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:76d6d900e5f48ce85edc919a0787e69337c660db8693ca5b516edd94374b8f10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chjiao/TAR-VIR
  tools:
  - Bowtie2
  - PEHaplo
  - SGA
  - SPAdes
  - TAR-VIR
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# viral-haplotype-reconstruction

## Summary

Reconstructs viral strain haplotypes de novo from metagenomic reads by combining overlap-extension read recruiting (TAR-VIR) with paired-end overlap-graph haplotype assembly (PEHaplo), without requiring complete or high-quality reference genomes.

## When to use

Use when a viral metagenomic read set (e.g. ART-illumina MiSeq 250 bp paired-end reads) must be classified against a partial or remotely related reference and resolved into strain-level haplotype contigs without a high-quality reference genome.

## When NOT to use

- The target virus shares no conservation with any reference genome — TAR-VIR's read recruiting will not detect a completely new virus
- Only generic species-level abundance profiling is needed and strain-level haplotypes are not of interest (generic assemblers suffice)
- A complete high-quality reference genome exists for the exact strain, making reference-based haplotype reconstruction tools more appropriate

## Inputs

- viral metagenomic paired-end read set (FASTA/FASTQ)
- partial or remotely related reference genome/gene sequence (e.g. bat coronavirus NC_014470.1)
- SAM file from Bowtie2 alignment with error function 'L,0,-0.6'
- error-corrected reads (karect)
- optional: known haplotype references (e.g. NC_004718.3-derived SARS-CoV haplotypes) for MetaQUAST evaluation

## Outputs

- recruited_reads.fa (enriched target-virus read set from overlap extension)
- Contigs.fa / Contigs_clipped.fa (PEHaplo haplotype contigs)
- comparative assembly metrics table (e.g. sars_cov_assembly_comparison.tsv: N50, genome coverage, mismatch rate per assembler)

## How to apply

First preprocess reads with karect error correction, then align them to a partial or remotely related reference with Bowtie2 using the error function 'L,0,-0.6' to produce a SAM file of seed reads. Run the TAR-VIR overlap-extension module (./build to index reads.fa, then ./overlap with a -c overlap cutoff set longer than half the read size, e.g. -c 180 for 250 bp reads) to recruit reads into recruited_reads.fa. Assemble the recruited reads with PEHaplo, using its '-f' option for a single fasta with mixed single- and paired-end reads, supplying overlap thresholds (e.g. -l 180, -l1 210), read length 250, insert size 600 and std 150; PEHaplo builds a paired-end overlap graph and greedily reconstructs haplotype contigs without any reference. Optionally compare PEHaplo against baselines (SGA, SPAdes, SAVAGE) on the identical recruited read set and evaluate contigs with MetaQUAST against known haplotypes (e.g. five SARS-CoV haplotypes from NC_004718.3).

## Related tools

- **TAR-VIR** (overlap-extension module that recruits target-viral reads from the SAM seed alignments into recruited_reads.fa using an overlap cutoff) — https://github.com/chjiao/TAR-VIR
- **PEHaplo** (de novo haplotype reconstruction from recruited reads via paired-end overlap graph and greedy path search; accepts a single mixed-read fasta via '-f') — github.com/chjiao/PEHaplo
- **Bowtie2** (aligns reads to the partial reference to generate seed-read SAM input; recommended error function 'L,0,-0.6' (sweep included 'L,0,-0.9'))
- **SPAdes** (baseline generic de novo assembler run on the identical recruited read set for comparison)
- **SGA** (baseline string-graph assembler used by PEHaplo internally and as an external comparison assembler)

## Examples

```
python pehaplo.py -f1 virus_1.fa -f2 virus_2.fa -l 180 -l1 210 -r 250 -F 600 -std 150 -n 3 -correct yes
```

## Evaluation signals

- On recruited SARS-CoV reads, PEHaplo should yield higher N50 and genome coverage than SGA, SPAdes and SAVAGE (reported: 7 contigs, N50 9,676, 98.9% genome coverage, 0.0% mismatch rate under Bowtie2 'L,0,-0.9')
- All recruited reads should originate from the target virus with no contamination from other simulated viruses (e.g. no Influenza or HBV reads)
- Assembled contigs should cover all five simulated SARS-CoV haplotypes when evaluated with MetaQUAST against NC_004718.3-derived haplotypes
- In the README toy example, overlap extension on 287 aligned HIV-1 seed reads should recruit 8008 reads (./overlap ... -c 180)
- For the HCV evaluation, only genotypes with at least 50 mapped reads should be assessed to keep MetaQUAST results reliable

## Limitations

- A completely new virus that does not share any conservation with any reference genome won't be detected by this method
- No software version numbers are provided for TAR-VIR, Bowtie2, BWA, or PEHaplo, complicating reproducibility
- No quantitative criteria are given for when local-mode screening via alignment length and score should accept or reject reads
- BWA can align slightly more reads than Bowtie2 with 'L,0,-0.9' (52,688 vs. 52,377), but the Bowtie2-recruited reads cover 20%-30% more genomes, so assembler choice and mapping parameters interact

## Evidence

- [abstract] we developed a hybrid pipeline named TAR-VIR that reconstructs viral strains without relying on complete or high-quality reference genomes: "we developed a hybrid pipeline named TAR-VIR that reconstructs viral strains without relying on complete or high-quality reference genomes"
- [methods] we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads: "we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads"
- [readme] The recruited reads usually contain both single- or paired-end reads, use the '-f' option of PEHaplo to input one fasta file.: "The recruited reads usually contain both single- or paired-end reads, use the '-f' option of PEHaplo to input one fasta file."
- [results] Worth noting is all the recruited reads are from SARS-CoV (no contamination from Influenza and HBV).: "Worth noting is all the recruited reads are from SARS-CoV (no contamination from Influenza and HBV)."
- [intro] A completely new virus that does not share any conservation with any reference genome won't be detected by our method.: [evidence span withheld — anchor status: not_found]
