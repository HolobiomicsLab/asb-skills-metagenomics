---
name: metagenomic-assembly-evaluation
description: Use when multiple de novo assemblers have been run on the same recruited
  viral read set and you must rank their contigs against known haplotypes with MetaQUAST
  metrics such as N50 and genome coverage.
when_to_use_negative:
- No ground-truth haplotypes or reference genome exist for the target virus, so MetaQUAST
  completeness and coverage cannot be computed
- The input is raw metagenomic reads that have not yet been classified/recruited;
  use the read-recruiting stage first rather than evaluating assemblies
- Only a single assembler was run and no comparison against baselines is intended
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3174
tools:
- name: PEHaplo
  role: TAR-VIR's default strain-level de novo assembler; builds a paired-end overlap
    graph and greedy path search to reconstruct haplotypes from recruited reads without
    a reference
  repo: https://github.com/chjiao/PEHaplo
- name: TAR-VIR
  role: Pipeline whose overlap-extension component produces the recruited_reads.fa
    input that is assembled and benchmarked
  repo: https://github.com/chjiao/TAR-VIR
- name: SPAdes
  role: Baseline generic de novo assembler run on the identical recruited read set
    for comparison
- name: SGA
  role: Baseline string-graph assembler run on the identical recruited read set for
    comparison (also a PEHaplo dependency)
- name: SAVAGE
  role: Baseline viral quasispecies assembler run on the identical recruited read
    set for comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:8d9828177685a105a059d26b376c4d3eac70f83f862ce3625d2c42fb92dbfec2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chjiao/PEHaplo
  tools:
  - PEHaplo
  - SAVAGE
  - SGA
  - SPAdes
  - TAR-VIR
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# metagenomic-assembly-evaluation

## Summary

Benchmark strain-level viral haplotype assemblies (e.g. TAR-VIR/PEHaplo contigs versus SGA, SPAdes, SAVAGE) with MetaQUAST against known reference-derived haplotypes, using N50, genome coverage and mismatch rate as the comparative metrics.

## When to use

Use when multiple de novo assemblers have been run on the same recruited viral read set and you must rank their contigs against known haplotypes with MetaQUAST metrics such as N50 and genome coverage.

## When NOT to use

- No ground-truth haplotypes or reference genome exist for the target virus, so MetaQUAST completeness and coverage cannot be computed
- The input is raw metagenomic reads that have not yet been classified/recruited; use the read-recruiting stage first rather than evaluating assemblies
- Only a single assembler was run and no comparison against baselines is intended

## Inputs

- Recruited viral read set in FASTA format (e.g. recruited_reads.fa containing mixed single- and paired-end reads)
- Contig sets (FASTA) from each assembler: PEHaplo, SPAdes, SAVAGE, SGA
- Reference genome from which haplotypes were derived (e.g. NC_004718.3) and the simulated haplotype sequences
- Read-length and paired-end insert-size parameters (e.g. 250 bp reads, 600 bp insert, 150 bp SD)

## Outputs

- Per-assembler MetaQUAST metrics table (number of contigs, N50, genome coverage, mismatch rate) compiled as e.g. sars_cov_assembly_comparison.tsv
- Contigs.fa produced by each assembler covering part or all of the target haplotypes

## How to apply

Assemble the identical recruited read set (e.g. recruited_reads.fa from TAR-VIR's overlap-extension step) with each assembler: PEHaplo via its '-f' option for a single fasta of mixed single- and paired-end reads, plus the baselines SPAdes, SAVAGE and SGA. For PEHaplo, supply overlap thresholds (e.g. -l 180 and -l1 210), read length 250, insert size 600 and standard deviation 150, matching the simulated MiSeq reads. Evaluate every contig set with MetaQUAST against the reference-derived haplotypes (five SARS-CoV haplotypes created from NC_004718.3), recording haplotype coverage, N50, number of contigs and mismatch rate per assembler. Compile the per-assembler metrics into a comparative table (e.g. sars_cov_assembly_comparison.tsv) and judge the strain-reconstruction pipeline successful when its contigs cover all five simulated haplotypes.

## Related tools

- **PEHaplo** (TAR-VIR's default strain-level de novo assembler; builds a paired-end overlap graph and greedy path search to reconstruct haplotypes from recruited reads without a reference) — https://github.com/chjiao/PEHaplo
- **TAR-VIR** (Pipeline whose overlap-extension component produces the recruited_reads.fa input that is assembled and benchmarked) — https://github.com/chjiao/TAR-VIR
- **SPAdes** (Baseline generic de novo assembler run on the identical recruited read set for comparison)
- **SGA** (Baseline string-graph assembler run on the identical recruited read set for comparison (also a PEHaplo dependency))
- **SAVAGE** (Baseline viral quasispecies assembler run on the identical recruited read set for comparison)

## Examples

```
python pehaplo.py -f1 virus_1.fa -f2 virus_2.fa -l 180 -l1 210 -r 250 -F 600 -std 150 -n 3 -correct yes
```

## Evaluation signals

- PEHaplo contigs cover all five simulated SARS-CoV haplotypes derived from NC_004718.3, matching the paper's reported complete assembly results
- Under the Bowtie2 'L,0,-0.9' seed set, PEHaplo yields 7 contigs, N50 9,676, 98.9% genome coverage and 0.0% mismatch rate
- PEHaplo's N50 and genome coverage exceed those of SGA, SPAdes and SAVAGE on the same recruited read set
- Mismatch rate against the reference haplotypes is 0.0% (or near zero) for a correct assembly
- All recruited reads are from the target virus with no contamination (e.g. no Influenza or HBV reads), so assembly metrics are not confounded

## Limitations

- MetaQUAST-style evaluation requires known reference-derived haplotypes; it cannot certify assembly quality for completely novel viruses
- The evaluation depends on the upstream read-recruiting parameters (e.g. Bowtie2 error function 'L,0,-0.9' recruited 52,377 reads covering 20%-30% more genome than BWA's 52,688 recruited reads), so rankings may shift with the seed set
- No software version numbers are provided for any of the compared tools (TAR-VIR, Bowtie2, BWA, PEHaplo), limiting reproducibility

## Evidence

- [results] On the recruited SARS-CoV reads, PEHaplo consistently achieved higher N50 and genome coverage: [evidence span withheld — anchor status: not_found]
- [results] The SARS-CoV haplotypes were created from the SARS-CoV reference (NC_004718.3) genome by mutating bases at randomly selected locations.: "The SARS-CoV haplotypes were created from the SARS-CoV reference (NC_004718.3) genome by mutating bases at randomly selected locations."
- [readme] PEHaplo does not need any reference genomes: [evidence span withheld — anchor status: external_doc]
- [readme] The recruited reads usually contain both single- or paired-end reads, use the '-f' option of PEHaplo: "The recruited reads usually contain both single- or paired-end reads, use the '-f' option of PEHaplo to input one fasta file."
