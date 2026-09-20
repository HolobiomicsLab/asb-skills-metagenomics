---
name: assembler-runtime-and-contig-count-profiling
description: Use when comparing the computational cost and assembly output (wall-clock
  time, total contigs, contigs >50 Kb) of a targeted assembly pipeline against complete
  metagenomic assembly or alternative assemblers on identical raw sequencing reads.
when_to_use_negative:
- Input is already assembled contigs — the benchmark requires raw unassembled FASTQ
  reads so all assemblers start from the same point
- The goal is gene abundance quantification across samples rather than assembler speed/output
  profiling
- 'Datasets are small isolate genomes where targeted assembly shows little runtime
  advantage (e.g., E. coli KLY was faster by complete assembly: 17.4 vs 23.3 min)'
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: CasCollect
  role: Targeted assembler benchmarked for runtime and cas operon recovery; run in
    protein mode with --meta flag
  repo: https://github.com/sandialabs/CasCollect
- name: metaSPAdes
  role: Assembler used for the complete-assembly baseline and internally by CasCollect
    for metagenomes
- name: VSEARCH
  role: Read-subset expansion within CasCollect at 95% identity over five cycles
  repo: https://github.com/torognes/vsearch
- name: HMMER
  role: Seed generation by matching reads against 120 Cas protein family profile HMMs
- name: bbduk
  role: Trimming and cleaning of reads before seed generation
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: CRISPRCasFinder
  role: Annotation/CRISPR array detection on targeted assembly contigs
- name: PRICE
  role: Alternative assembler benchmarked for runtime and full cas operon spanning
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:045679e9f5a1b5d5b6e30606d06a161d94d16deca9940a01d7c9dc29d6bb7c70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - BBTools
  - CRISPRCasFinder
  - CasCollect
  - HMMER
  - PRICE
  - VSEARCH
  - bbduk
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# assembler-runtime-and-contig-count-profiling

## Summary

Profiles the wall-clock runtime and contig yield of a targeted assembler (CasCollect) against complete metagenomic assembly plus annotation and a rolling-circle-style assembler (PRICE) on the same paired-end FASTQ datasets, quantifying the speed advantage of targeted over whole-sample assembly.

## When to use

Use when comparing the computational cost and assembly output (wall-clock time, total contigs, contigs >50 Kb) of a targeted assembly pipeline against complete metagenomic assembly or alternative assemblers on identical raw sequencing reads.

## When NOT to use

- Input is already assembled contigs — the benchmark requires raw unassembled FASTQ reads so all assemblers start from the same point
- The goal is gene abundance quantification across samples rather than assembler speed/output profiling
- Datasets are small isolate genomes where targeted assembly shows little runtime advantage (e.g., E. coli KLY was faster by complete assembly: 17.4 vs 23.3 min)

## Inputs

- Paired-end metagenomic FASTQ files (e.g., SRR10598175, ERR472738, SRR11195315, SRR5823773 from SRA/ENA)
- CasCollect default Cas protein profile HMM set (120 HMMs)
- Contaminant reference for cleaning (default: human genome)

## Outputs

- Wall-clock runtimes per assembler per dataset (CasCollect vs complete assembly + annotation vs PRICE)
- Mean and standard deviation of runtimes across datasets
- Total contig counts and counts of contigs >50 Kb per approach
- PRICE outcome assessment (runtime; whether contigs span entire cas operons)
- metagenome_benchmark_summary.csv

## How to apply

Run both pipelines on the identical raw paired-end FASTQ inputs: the targeted tool (CasCollect in default protein mode with the --meta flag, metaSPAdes in place of SPAdes, default settings of 120 Cas protein HMMs, five vsearch read-subset expansion cycles at 95% identity, 1 cpu and 20 Gb RAM) and the complete-assembly baseline (metaSPAdes followed by automated Perl annotation with a 500 bp contig size cutoff). Record wall-clock runtimes for both approaches per dataset, then compute means and standard deviations across datasets to express the targeted method's fraction of baseline cost (here ~3%, 1.7 ± 0.3 h vs 50.6 ± 3.0 h). Separately count total contigs and contigs longer than 50 Kb produced by each approach to compare output yield. Optionally run a third assembler (PRICE) on the same reads, recording its runtime and whether its contigs span entire cas operons. Assemble runtimes, contig counts, and qualitative outcomes (e.g., full-operon recovery) into a summary CSV.

## Related tools

- **CasCollect** (Targeted assembler benchmarked for runtime and cas operon recovery; run in protein mode with --meta flag) — https://github.com/sandialabs/CasCollect
- **metaSPAdes** (Assembler used for the complete-assembly baseline and internally by CasCollect for metagenomes)
- **VSEARCH** (Read-subset expansion within CasCollect at 95% identity over five cycles) — https://github.com/torognes/vsearch
- **HMMER** (Seed generation by matching reads against 120 Cas protein family profile HMMs)
- **bbduk** (Trimming and cleaning of reads before seed generation) — https://jgi.doe.gov/data-and-tools/bbtools/
- **CRISPRCasFinder** (Annotation/CRISPR array detection on targeted assembly contigs)
- **PRICE** (Alternative assembler benchmarked for runtime and full cas operon spanning)

## Examples

```
CasCollect.py -1 SRR10598175_1.fq.gz -2 SRR10598175_2.fq.gz --meta
```

## Evaluation signals

- Wall-clock timings recorded for both pipelines on identical reads; targeted runtime expressed as a fraction of baseline (expected ~3% on real metagenomes)
- Runtime means and standard deviations reported across all four datasets (1.7 ± 0.3 h vs 50.6 ± 3.0 h)
- Contig counts tabulated both as totals and restricted to contigs >50 Kb for each approach
- Complete-assembly annotation applied a consistent 500 bp contig size cutoff (average ~161K contigs analyzed)
- PRICE output assessed qualitatively for contigs spanning entire cas operons

## Limitations

- Runtimes are hardware- and configuration-dependent (CasCollect run with 1 cpu and 20 Gb RAM); comparisons are only valid under matched resources
- A 500 bp contig size cutoff for annotation reduces the contigs analyzed and shapes the contig-count comparison
- On small isolate genomes, targeted assembly can be slower than complete assembly (e.g., E. coli KLY: 23.3 vs 17.4 min), so the runtime advantage is dataset-size dependent
- No software version numbers or accession IDs are given for the tools or datasets, limiting reproducibility

## Evidence

- [results] CasCollect required on average only 3% of the time of complete assembly: [evidence span withheld — anchor status: not_found]
- [results] pipeline relies on third-party programs: "This pipeline relies on several third-party programs that include: HMMER (21), VSEARCH (22), SPAdes (23) and CRISPRCasFinder (13)"
- [results] default expansion parameters: "The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command."
- [results] annotation contig cutoff: "Our Perl script for automating the annotation had set a size cutoff of 500 bp, which reduced the total number of contigs analyzed for the complete assembly to an average of 161K contigs"
- [readme] README dependency check and citation: "This pipeline relies on python and perl as well as requiring the installation of several pieces of software within the users path"
- [results] metagenome mode: "For metagenomes, metaSPAdes is used in-place-of SPAdes."
