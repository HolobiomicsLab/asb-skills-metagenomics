---
name: quast-assembly-quality-metrics
description: Use when comparing contig sets from multiple metagenome assemblers against
  a known reference (strain genomes) and you need Quast metrics such as genome fraction,
  N50, NGA50 and Error Rate for benchmark tables.
when_to_use_negative:
- When no reference genomes are available for the strains in the metagenome — Quast
  genome fraction requires a reference to align contigs against
- When comparing long-read (third-generation) assemblies, since the article notes
  the high error rate of third-generation reads remains unresolved and short Illumina
  reads constrain contig length
- When the goal is read-level QC rather than assembly-contig quality assessment
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: Quast
  role: Computes genome fraction, NGA50, N50, Error Rate, misassembled contig rate
    and N rate for each contig set against the strain-genome reference
- name: metaQuast
  role: Evaluates quality of assemblies of the 10 spiked-in Salmonella strains
- name: StrainXpress
  role: Produces final_contigs.fasta contig sets (run with python strainxpress.py
    -fq all_reads.fq, -fast for large datasets) that are evaluated against competitor
    tools
  repo: https://github.com/HaploKit/StrainXpress
- name: SPAdes
  role: Competitor de novo assembler whose Quast metrics form the second-highest baseline
    (74.74% genome fraction on low-complexity 2x250 bp data)
- name: IDBA-UD
  role: Competitor assembler; ranked first on identity (99.91%) in comparisons
- name: Minia3
  role: Competitor assembler contributing baseline genome-fraction values in real-data
    benchmarks
- name: MEGAHIT
  role: Competitor assembler that outperforms all other methods in contig length (N50)
provenance:
  source_task_ids:
  - task_004
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:4c5b5b7ccf9b0eb946a1991b84aa5ff1a80250927afa793884715e134d97d47a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - GATB-Minia
  - IDBA-UD
  - MEGAHIT
  - MetaSPAdes
  - Minia3
  - Quast
  - SPAdes
  - StrainXpress
  - metaQuast
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# quast-assembly-quality-metrics

## Summary

Evaluate strain-aware metagenome assemblies (StrainXpress vs SPAdes, IDBA-UD, Minia3, MEGAHIT, GATB-Minia) with Quast/metaQuast, recording genome fraction, NGA50, N50, Error Rate, misassembled contig rate and N rate per assembler-dataset combination.

## When to use

Use when comparing contig sets from multiple metagenome assemblers against a known reference (strain genomes) and you need Quast metrics such as genome fraction, N50, NGA50 and Error Rate for benchmark tables.

## When NOT to use

- When no reference genomes are available for the strains in the metagenome — Quast genome fraction requires a reference to align contigs against
- When comparing long-read (third-generation) assemblies, since the article notes the high error rate of third-generation reads remains unresolved and short Illumina reads constrain contig length
- When the goal is read-level QC rather than assembly-contig quality assessment

## Inputs

- Contig FASTA files per assembler (e.g. stageb/final_contigs.fasta from StrainXpress)
- Concatenated strain-genome reference FASTA (CAMISIM ground truth, Bmock12, NWCs)
- StrainEst-derived E. coli ground truth for Gut Metagenome datasets
- Interleaved FASTQ read sets (SRX4901583, PRJNA379884, CAMISIM simulated data)

## Outputs

- quast_benchmark_table.csv with 6 Quast metrics for every assembler x dataset combination
- real_data_benchmark.csv with one row per assembler-dataset combination
- gain_summary.csv with per-dataset genome-fraction gains and mean/Q1/median/Q3 summary

## How to apply

Run Quast on every assembler's contig set (e.g. StrainXpress's stageb/final_contigs.fasta) against the concatenated strain-genome reference; for spike-in experiments use metaQuast on the 10 spiked-in Salmonella strains. Record genome fraction, NGA50, N50, Error Rate, misassembled contig rate and N rate per assembler per dataset (9 CAMISIM datasets, Bmock12, NWCs, 5 Gut Metagenome samples). For Gut Metagenome, assemble each of the five datasets individually and evaluate E. coli contigs against the StrainEst-derived ground truth. To quantify strain-specific sequence gain, compute per-dataset (StrainXpress_genome_fraction - best_competitor_genome_fraction) / best_competitor_genome_fraction * 100 and aggregate mean, Q1, median, Q3. Validate against reported values: 93.45% vs 74.74% genome fraction (low-complexity 2x250 bp) and gain summary of 26.75% mean (Q1 18.51%, median 26.60%, Q3 35.05%).

## Related tools

- **Quast** (Computes genome fraction, NGA50, N50, Error Rate, misassembled contig rate and N rate for each contig set against the strain-genome reference)
- **metaQuast** (Evaluates quality of assemblies of the 10 spiked-in Salmonella strains)
- **StrainXpress** (Produces final_contigs.fasta contig sets (run with python strainxpress.py -fq all_reads.fq, -fast for large datasets) that are evaluated against competitor tools) — https://github.com/HaploKit/StrainXpress
- **SPAdes** (Competitor de novo assembler whose Quast metrics form the second-highest baseline (74.74% genome fraction on low-complexity 2x250 bp data))
- **IDBA-UD** (Competitor assembler; ranked first on identity (99.91%) in comparisons)
- **Minia3** (Competitor assembler contributing baseline genome-fraction values in real-data benchmarks)
- **MEGAHIT** (Competitor assembler that outperforms all other methods in contig length (N50))

## Examples

```
python strainxpress.py -fq all_reads.fq -fast; quast.py stageb/final_contigs.fasta -r strains_reference.fa
```

## Evaluation signals

- On low-complexity 2x250 bp CAMISIM data, StrainXpress genome fraction should be 93.45% with SPAdes at 74.74% as the second-highest de novo assembler
- Per-dataset StrainXpress gains over the best state-of-the-art assembler should aggregate to mean 26.75%, Q1 18.51%, median 26.60%, Q3 35.05%
- quast_benchmark_table.csv must contain all six Quast metrics for every assembler x dataset combination (6 assemblers x 9 CAMISIM datasets)
- On Bmock12, StrainXpress genome fraction should exceed all other assemblers by at least 3.7% and approach 99.04% overall
- Quast identity values should fall near 99.9% (StrainXpress 99.93% vs IDBA-UD 99.91%)

## Limitations

- Short length of Illumina-type reads constrains contig length, preventing strain-specific contigs that span entire genomes, so N50-based comparisons can favor MEGAHIT
- No specific version numbers are stated for StrainXpress or any compared tool, a reproducibility gap for the benchmark
- metaQuast is only applied to the spike-in Salmonella experiments; other datasets are evaluated with plain Quast

## Evidence

- [results] Quantities referring to the criteria listed in the corresponding Tables were determined using Quast (40): "Quantities referring to the criteria listed in the corresponding Tables were determined using Quast (40)"
- [results] metaQuast on spike-ins: "evaluated the quality of the assemblies of the 10 spiked in salmonella strains using metaQuast"
- [results] Genome fraction result: "StrainXpress: 93.45% versus 74.74% by SPAdes, as the second highest scoring"
- [abstract] Gain summary: "The amount of reconstructed strain-specific sequence exceeds that of the current state-of-the-art approaches by on average 26.75% across all data sets"
- [readme] README invocation: "python ../scripts/strainxpress.py -fq all_reads.fq"
- [results] Gut Metagenome per-dataset assembly: "Reads were assembled for each of the five data sets individually"
