---
name: paired-end-overlap-graph-assembly
description: Use when you have a FASTA of mixed single- and paired-end viral reads
  (e.g. TAR-VIR recruited_reads.fa) and need to reconstruct strain-level haplotypes
  de novo, without a complete reference genome.
when_to_use_negative:
- You only need generic de novo assembly of non-strain-resolved contigs (e.g. SPAdes-style
  whole-genome assembly) rather than haplotype reconstruction from recruited reads
- No paired-end information is available, since PEHaplo relies on paired-end read
  connections to resolve haplotypes
- The target virus shares no conservation with any reference, so the upstream read-recruiting
  stage that generates PEHaplo's input cannot be performed
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: PEHaplo
  role: De novo strain-level haplotype assembler that builds a paired-end overlap
    graph, merges linked cliques and applies greedy path search on the recruited reads
  repo: https://github.com/chjiao/PEHaplo
- name: TAR-VIR
  role: Upstream overlap-extension read-recruiting pipeline whose recruited_reads.fa
    feeds PEHaplo; TAR-VIR's default assembler is PEHaplo
  repo: https://github.com/chjiao/TAR-VIR
- name: SPAdes
  role: Generic de novo baseline assembler run on the identical recruited read set
    for benchmarking PEHaplo's output with MetaQUAST
- name: SGA
  role: String-graph assembler used as an assembly baseline and also required in PEHaplo's
    path for alignment-based contig correction
- name: Karect
  role: Default error-correction tool for reads before assembly (recommended preprocessing
    in TAR-VIR; also used inside PEHaplo)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:35857a03addd90634630d90e304d070065130c77c449b6d355cd6b63deb17561
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chjiao/PEHaplo
  tools:
  - Karect
  - PEHaplo
  - SGA
  - SPAdes
  - TAR-VIR
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# paired-end-overlap-graph-assembly

## Summary

De novo strain-level assembly of viral haplotypes from recruited reads using PEHaplo, which builds a paired-end overlap graph and merges linked cliques to reconstruct haplotypes without any reference genome. In TAR-VIR this is the second stage applied after overlap-extension read recruitment.

## When to use

Use when you have a FASTA of mixed single- and paired-end viral reads (e.g. TAR-VIR recruited_reads.fa) and need to reconstruct strain-level haplotypes de novo, without a complete reference genome.

## When NOT to use

- You only need generic de novo assembly of non-strain-resolved contigs (e.g. SPAdes-style whole-genome assembly) rather than haplotype reconstruction from recruited reads
- No paired-end information is available, since PEHaplo relies on paired-end read connections to resolve haplotypes
- The target virus shares no conservation with any reference, so the upstream read-recruiting stage that generates PEHaplo's input cannot be performed

## Inputs

- Recruited viral reads in FASTA (TAR-VIR recruited_reads.fa) containing mixed single- and paired-end reads, or paired-end .1/.2 FASTA files
- Overlap threshold(s), read length, insert size and insert standard deviation (e.g. 180/210, 250, 600, 150)

## Outputs

- Contigs.fa — produced contigs that are part of or full viral haplotypes
- PEG_nodes_sequences.fa — node sequences in the paired-end overlap graph
- Contigs_clipped.fa — contigs after error correction when -correct yes

## How to apply

Run PEHaplo on the recruited reads using its '-f' option when single- and paired-end reads are in one fasta file, or '-f1/-f2' for separate paired-end files. Set the overlap threshold used for overlap-graph construction (e.g. -l 180) and the post-clique-merging cutoff (-l1 210, i.e. longer than half the read length to prevent error overlaps), plus read length (250 bp), insert size (600 bp) and its standard deviation (150), duplication threshold (-n 3) and optionally alignment-based contig correction (-correct yes) which uses karect, SGA and Bowtie2. Optionally pre-run error correction with karect, which is TAR-VIR's default. The rationale is that paired-end connections link cliques in the overlap graph so full viral haplotypes can be recovered even when haplotypes are remotely related to characterized references.

## Related tools

- **PEHaplo** (De novo strain-level haplotype assembler that builds a paired-end overlap graph, merges linked cliques and applies greedy path search on the recruited reads) — https://github.com/chjiao/PEHaplo
- **TAR-VIR** (Upstream overlap-extension read-recruiting pipeline whose recruited_reads.fa feeds PEHaplo; TAR-VIR's default assembler is PEHaplo) — https://github.com/chjiao/TAR-VIR
- **SPAdes** (Generic de novo baseline assembler run on the identical recruited read set for benchmarking PEHaplo's output with MetaQUAST)
- **SGA** (String-graph assembler used as an assembly baseline and also required in PEHaplo's path for alignment-based contig correction)
- **Karect** (Default error-correction tool for reads before assembly (recommended preprocessing in TAR-VIR; also used inside PEHaplo))

## Examples

```
python ../pehaplo.py -f1 ../raw_test_data/virus_1.fa -f2 ../raw_test_data/virus_2.fa -l 180 -l1 210 -r 250 -F 600 -std 150 -n 3 -correct yes
```

## Evaluation signals

- Against the five SARS-CoV haplotypes from NC_004718.3, PEHaplo assembly covering all haplotypes with higher N50 and genome coverage than SGA, SPAdes and SAVAGE baselines
- Benchmark comparison table (sars_cov_assembly_comparison.tsv) records 7 contigs, N50 9,676, 98.9% genome coverage and 0.0% mismatch rate under the Bowtie2 'L,0,-0.9' seed set
- PEHaplo test-data run on processed_test_data should complete the apsp_overlap_clique.py workflow and emit Contigs.fa and PEG_nodes_sequences.fa
- Recruited-read input purity: all recruited reads from the target virus (no contamination), e.g. all SARS-CoV with no Influenza/HBV reads

## Limitations

- A completely new virus sharing no conservation with any reference genome will not be detected, so no reads reach this assembly stage
- No software version numbers are provided for TAR-VIR or PEHaplo, complicating reproducibility
- The overlap threshold should be set longer than half of the read size to prevent recruiting chimeric reads; no specific recommended value is stated in the discussion
- PEHaplo requires Python 2.7.x with networkx 1.11 and auxiliary tools (Apsp, Readjoiner, SGA, Samtools, karect) in the path, a legacy dependency stack

## Evidence

- [readme] PEHaplo is a de novo assembly tool for recovering virus haplotypes from virus quasispecies sequencing data. It utilizes overlap graph and paired-end information to recover virus haplotypes.: [evidence span withheld — anchor status: external_doc]
- [intro] de novo assembly of viral haplotypes from recruited reads with PEHaplo, which is a haplotype reconstruction tool: "(2) de novo assembly of viral haplotypes from recruited reads with PEHaplo, which is a haplotype reconstruction tool"
- [methods] we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads: "we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads"
- [readme] PEHaplo does not need any reference genomes and thus can be applied for identifying new haplotyps or haplotypes that are remotely related to characterized ones.: [evidence span withheld — anchor status: external_doc]
- [readme] The recruited reads usually contain both single- or paired-end reads, use the '-f' option of PEHaplo to input one fasta file.: "The recruited reads usually contain both single- or paired-end reads, use the '-f' option of PEHaplo to input one fasta file."
