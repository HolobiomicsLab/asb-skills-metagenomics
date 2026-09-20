---
name: multi-assembler-benchmark-table-compilation
description: Use when comparing the assembly quality of several de novo assemblers
  across a common set of metagenome datasets and you need a single per-assembler x
  per-dataset table of Quast metrics.
when_to_use_negative:
- The data are real reference-based (not de novo) assemblies requiring reference-assisted
  evaluation against a single reference genome rather than strain-level de novo benchmarking
- You need a single assembler optimized rather than a comparative benchmark — MEGAHIT,
  for instance, wins on contig length (N50) while StrainXpress wins on genome fraction,
  so the table is not a single-tool tuning guide
- The input is long high-error-rate third-generation reads, which the paper reports
  remain unresolved for this strain-aware assembly workflow
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: StrainXpress
  role: Primary strain-aware de novo assembler benchmarked; run with strainxpress.py
    -fq on interleaved FASTQ, output final_contigs.fasta from stageb folder
  repo: https://github.com/HaploKit/StrainXpress
- name: SPAdes
  role: Competitor de novo assembler; second highest genome fraction (74.74%) on low-complexity
    2x250 bp data
- name: MetaSPAdes
  role: Competitor de novo assembler run on the same nine CAMISIM datasets
- name: IDBA-UD
  role: Competitor de novo assembler; top identity score (99.91%) among non-StrainXpress
    tools
- name: GATB-Minia
  role: Competitor de novo assembler (Minia3 family)
- name: MEGAHIT
  role: Competitor de novo assembler; outperforms all others in contig length (N50)
- name: Quast
  role: Evaluation tool computing genome fraction, NGA50, N50, error rate, misassembled
    contig rate and N rate against the strain-genome reference
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:c0b0024d438fc0e271b8885b117a3b7abc79a214ec5b992e714e9e7fb8023056
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - GATB-Minia
  - IDBA-UD
  - MEGAHIT
  - MetaSPAdes
  - Quast
  - SPAdes
  - StrainXpress
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# multi-assembler-benchmark-table-compilation

## Summary

Benchmark multiple de novo metagenome assemblers (e.g. StrainXpress, SPAdes, MetaSPAdes, IDBA-UD, GATB-Minia, MEGAHIT) on a shared panel of CAMISIM-simulated datasets and aggregate Quast metrics (genome fraction, NGA50, N50, error rate, misassembly rate, N rate) into one comparison table.

## When to use

Use when comparing the assembly quality of several de novo assemblers across a common set of metagenome datasets and you need a single per-assembler x per-dataset table of Quast metrics.

## When NOT to use

- The data are real reference-based (not de novo) assemblies requiring reference-assisted evaluation against a single reference genome rather than strain-level de novo benchmarking
- You need a single assembler optimized rather than a comparative benchmark — MEGAHIT, for instance, wins on contig length (N50) while StrainXpress wins on genome fraction, so the table is not a single-tool tuning guide
- The input is long high-error-rate third-generation reads, which the paper reports remain unresolved for this strain-aware assembly workflow

## Inputs

- Interleaved FASTQ files for nine CAMISIM simulated metagenomes (low/medium/high complexity x three read lengths, e.g. 2x250 bp)
- Concatenated strain-genome reference FASTA for Quast evaluation
- Contig sets (final_contigs.fasta or equivalent) from each of the six assemblers per dataset

## Outputs

- quast_benchmark_table.csv containing genome fraction, NGA50, N50, error rate, misassembled contig rate, and N rate for every assembler x dataset combination (6 assemblers x 9 datasets)
- Per-assembler contig FASTA files (e.g. final_contigs.fasta from StrainXpress's stageb folder)

## How to apply

Run each assembler on the identical input FASTQ files — for StrainXpress use interleaved FASTQ via 'python strainxpress.py -fq all_reads.fq' (adding -fast for the large, high-complexity CAMISIM datasets) and take final_contigs.fasta from the stageb folder. Run IDBA-UD, SPAdes, MetaSPAdes, GATB-Minia and MEGAHIT on the same nine CAMISIM datasets (low/medium/high complexity x three read lengths) to obtain comparable contig sets. Evaluate every contig set with Quast against the concatenated strain-genome reference to compute genome fraction, NGA50, N50, error rate, misassembled contig rate and N rate. Aggregate all per-assembler, per-dataset metrics into quast_benchmark_table.csv. Sanity-check against published trends, e.g. StrainXpress genome fraction of 93.45% versus SPAdes at 74.74% on the low-complexity 2x250 bp data.

## Related tools

- **StrainXpress** (Primary strain-aware de novo assembler benchmarked; run with strainxpress.py -fq on interleaved FASTQ, output final_contigs.fasta from stageb folder) — https://github.com/HaploKit/StrainXpress
- **SPAdes** (Competitor de novo assembler; second highest genome fraction (74.74%) on low-complexity 2x250 bp data)
- **MetaSPAdes** (Competitor de novo assembler run on the same nine CAMISIM datasets)
- **IDBA-UD** (Competitor de novo assembler; top identity score (99.91%) among non-StrainXpress tools)
- **GATB-Minia** (Competitor de novo assembler (Minia3 family))
- **MEGAHIT** (Competitor de novo assembler; outperforms all others in contig length (N50))
- **Quast** (Evaluation tool computing genome fraction, NGA50, N50, error rate, misassembled contig rate and N rate against the strain-genome reference)

## Examples

```
python strainxpress.py -fq all_reads.fq && quast.py final_contigs.fasta -r strains_reference.fa
```

## Evaluation signals

- quast_benchmark_table.csv contains all six Quast metrics for every assembler x dataset combination (6 assemblers x 9 datasets) with no missing cells
- StrainXpress genome fraction on the low-complexity 2x250 bp dataset is ~93.45%, versus ~74.74% for SPAdes as the second-highest scorer
- StrainXpress identity metrics are consistent with 99.93% versus IDBA-UD's 99.91% trend on comparable datasets
- MEGAHIT shows the highest N50 values among assemblers in the table, matching the published contig-length trend
- Each contig set was produced from the identical input FASTQ per dataset, so the comparison is apples-to-apples

## Limitations

- No software version numbers are stated for any of the compared tools, which limits exact reproducibility of the benchmark
- Short length of Illumina-type reads constrains contig length, preventing full-length strain-specific contigs from being assembled
- High error rate of third-generation sequencing reads introduces unresolved issues for this assembly benchmarking approach

## Evidence

- [results] Genome fraction on low-complexity data: "StrainXpress: 93.45% versus 74.74% by SPAdes, as the second highest scoring"
- [results] Evaluation with Quast: "Quantities referring to the criteria listed in the corresponding Tables were determined using Quast (40)"
- [intro] Assembler selection: "A non-exhaustive selection of state-of-the-art approaches encompasses IDBA-UD (6), SPAdes (7), Minia3 (8) and MEGAHIT (9)."
- [readme] StrainXpress installation: "conda install -c bioconda python=3.6 scipy pandas minimap2"
- [readme] StrainXpress run command: "python ../scripts/strainxpress.py -fq all_reads.fq"
- [results] MEGAHIT N50 trend: "MEGAHIT outperforms all other methods in terms of contig length (N50 in particular)"
