---
name: cross-assembler-performance-tabulation
description: Use when comparing de novo metagenome assemblers on the same read sets
  and you need a consolidated table of Quast metrics (genome fraction, N50, identity)
  per assembler-dataset combination.
when_to_use_negative:
- Input is a single assembly that only needs quality reporting rather than cross-assembler
  comparison
- Reads are long third-generation reads — the paper states the high error rate of
  third-generation sequencing reads remains an unresolved issue
- Datasets are synthetic benchmark sets rather than the real Bmock12/NWCs/Gut Metagenome
  communities this tabulation covers
edam_operation: http://edamontology.org/operation_0544
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: StrainXpress
  role: Primary strain-aware assembler whose Quast metrics are tabulated; run via
    strainxpress.py on interleaved FASTQ
  repo: https://github.com/HaploKit/StrainXpress
- name: SPAdes
  role: Competing assembler run on the same read sets for comparison
- name: IDBA-UD
  role: Competing assembler run on the same read sets for comparison
- name: Minia3
  role: Competing assembler run on the same read sets for comparison
- name: MEGAHIT
  role: Competing assembler run on the same read sets for comparison
- name: Quast
  role: Evaluates all assemblies, producing genome fraction, N50 and identity metrics
    for the CSV table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:a4daf928aa3cf2a0ff0a7cb4d45f46150adadccd1114be4e101fba88162c0a1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - IDBA-UD
  - MEGAHIT
  - Minia3
  - Quast
  - SPAdes
  - StrainXpress
  edam_operation: http://edamontology.org/operation_0544
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Cross-assembler performance tabulation

## Summary

Benchmark StrainXpress against SPAdes, IDBA-UD, Minia3 and MEGAHIT on real metagenome datasets (Bmock12, NWCs, Gut Metagenome) and aggregate Quast-derived metrics (genome fraction, N50, identity) into a single per-assembler-per-dataset CSV table.

## When to use

Use when comparing de novo metagenome assemblers on the same read sets and you need a consolidated table of Quast metrics (genome fraction, N50, identity) per assembler-dataset combination.

## When NOT to use

- Input is a single assembly that only needs quality reporting rather than cross-assembler comparison
- Reads are long third-generation reads — the paper states the high error rate of third-generation sequencing reads remains an unresolved issue
- Datasets are synthetic benchmark sets rather than the real Bmock12/NWCs/Gut Metagenome communities this tabulation covers

## Inputs

- Interleaved FASTQ read sets for Bmock12 (SRX4901583), NWCs, and 5 Gut Metagenome patient samples (PRJNA379884)
- Assembly contigs (FASTA) from StrainXpress, SPAdes, IDBA-UD, Minia3 and MEGAHIT
- StrainEst-derived ground truth for the 11 E. coli strains in the Gut Metagenome samples

## Outputs

- real_data_benchmark.csv with one row per assembler-dataset combination
- Quast metrics per sample per assembler (genome fraction, N50, identity and related metrics)

## How to apply

Download the interleaved FASTQ read sets for each real dataset (Bmock12 SRX4901583, NWCs, and the 5 Gut Metagenome samples from PRJNA379884). Run StrainXpress with 'python strainxpress.py -fq <reads>.fq' (adding the -fast flag for larger datasets) and collect stageb/final_contigs.fasta; run SPAdes, IDBA-UD, Minia3 and MEGAHIT on the same read sets, optionally using StrainXpress's XC clustering as a pre-processing step. For the five Gut Metagenome datasets, assemble each individually and evaluate E. coli contigs against the StrainEst-derived ground truth. Evaluate all assemblies with Quast, recording genome fraction, N50, identity and related metrics, then aggregate into real_data_benchmark.csv with one row per assembler-dataset combination.

## Related tools

- **StrainXpress** (Primary strain-aware assembler whose Quast metrics are tabulated; run via strainxpress.py on interleaved FASTQ) — https://github.com/HaploKit/StrainXpress
- **SPAdes** (Competing assembler run on the same read sets for comparison)
- **IDBA-UD** (Competing assembler run on the same read sets for comparison)
- **Minia3** (Competing assembler run on the same read sets for comparison)
- **MEGAHIT** (Competing assembler run on the same read sets for comparison)
- **Quast** (Evaluates all assemblies, producing genome fraction, N50 and identity metrics for the CSV table)

## Examples

```
python strainxpress.py -fq all_reads.fq -fast; quast.py final_contigs.fasta spades.fasta megahit.fasta -o quast_out
```

## Evaluation signals

- real_data_benchmark.csv contains one row per assembler-dataset combination with no missing combinations (5 assemblers x 3 dataset groups)
- StrainXpress genome fraction on Bmock12 approaches 99.04% and exceeds all other assemblers by at least 3.7%
- Quast reports identity values around 99.9% for top performers (StrainXpress 99.93% versus IDBA-UD 99.91%)
- Each of the five Gut Metagenome datasets was assembled individually and E. coli contigs checked against the StrainEst-derived ground truth
- N50 values are recorded so MEGAHIT's expected advantage in contig length is visible in the table

## Limitations

- No software version numbers are stated for any of the compared tools, hampering exact reproduction
- Short length of Illumina-type reads constrains contig length, preventing full-length strain-specific contigs
- The high error rate of third-generation sequencing reads introduces issues that still need to be resolved

## Evidence

- [results] Real-data benchmark scope: [evidence span withheld — anchor status: not_found]
- [results] Quast as evaluation tool: "Quantities referring to the criteria listed in the corresponding Tables were determined using Quast (40)"
- [results] XC pre-processing option: "We considered the application of StrainXpress’s clustering procedure (referred to as ‘XC’ in Table 1) as a pre-processing step for the other methods"
- [results] Individual Gut Metagenome assembly: "Reads were assembled for each of the five data sets individually"
- [readme] README invocation and output location: "python ../scripts/strainxpress.py -fq all_reads.fq"
- [results] MEGAHIT N50 advantage: "MEGAHIT outperforms all other methods in terms of contig length (N50 in particular)"
