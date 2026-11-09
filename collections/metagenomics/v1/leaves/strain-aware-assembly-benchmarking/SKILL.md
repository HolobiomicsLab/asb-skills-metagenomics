---
name: strain-aware-assembly-benchmarking
description: Use when comparing de novo strain-aware metagenome assemblers on real
  short-read community datasets and need reproducible Quast genome-fraction, N50 and
  identity comparisons across assemblers.
when_to_use_negative:
- Reads are long third-generation sequencing data — the paper notes the high error
  rate of third-generation reads 'introduce a range of issues that still need to be
  resolved' and StrainXpress targets Illumina-type short reads
- The goal is a single-genome (isolate) assembly rather than separating multiple strains
  within a metagenome
- Input is already-assembled contigs and only evaluation is needed — use Quast/metaQuast
  directly instead of re-running the assembly benchmark
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: StrainXpress
  role: Strain-aware de novo metagenome assembler being benchmarked; run via 'python
    strainxpress.py -fq <reads>.fq'
  repo: https://github.com/HaploKit/StrainXpress
- name: SPAdes
  role: Competing state-of-the-art assembler used as comparison baseline
- name: IDBA-UD
  role: Competing state-of-the-art assembler used as comparison baseline
- name: Minia3
  role: Competing state-of-the-art assembler used as comparison baseline
- name: MEGAHIT
  role: Competing state-of-the-art assembler used as comparison baseline
- name: Quast
  role: Assembly quality evaluation (genome fraction, N50, identity) of all assemblies
- name: minimap2
  role: Dependency of StrainXpress used in its overlap/alignment stage
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:765136e2110dc0b565a715a78d074347e2c5acf8ca3a2d5f33ec3322a5c76d3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - IDBA-UD
  - MEGAHIT
  - Minia3
  - PEHaplo
  - Quast
  - SPAdes
  - StrainXpress
  - TAR-VIR
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- strain-level-de-novo-assembly-benchmarking
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# strain-aware-assembly-benchmarking

## Summary

Benchmark StrainXpress against state-of-the-art metagenome assemblers (SPAdes, IDBA-UD, Minia3, MEGAHIT) on real datasets (Bmock12, NWCs, Gut Metagenome) using Quast metrics such as genome fraction and N50.

## When to use

Use when comparing de novo strain-aware metagenome assemblers on real short-read community datasets and need reproducible Quast genome-fraction, N50 and identity comparisons across assemblers.

## When NOT to use

- Reads are long third-generation sequencing data — the paper notes the high error rate of third-generation reads 'introduce a range of issues that still need to be resolved' and StrainXpress targets Illumina-type short reads
- The goal is a single-genome (isolate) assembly rather than separating multiple strains within a metagenome
- Input is already-assembled contigs and only evaluation is needed — use Quast/metaQuast directly instead of re-running the assembly benchmark

## Inputs

- Interleaved FASTQ read sets for Bmock12 (SRX4901583)
- Interleaved FASTQ read sets for NWCs (natural whey starter cultures)
- Interleaved FASTQ read sets for 5 Gut Metagenome patient samples (PRJNA379884)
- StrainEst-derived ground truth for E. coli strains in Gut Metagenome

## Outputs

- final_contigs.fasta per assembler per dataset (stageb/final_contigs.fasta for StrainXpress)
- Quast metrics per sample per assembler (genome fraction, N50, identity)
- Aggregated real_data_benchmark.csv with one row per assembler-dataset combination

## How to apply

Download the interleaved FASTQ read sets for the three real datasets: Bmock12 mock community (SRX4901583), natural whey starter cultures (NWCs), and the 5 Gut Metagenome patient samples (PRJNA379884). Install StrainXpress in a conda environment (python=3.6, scipy, pandas, minimap2), compile with 'sh install.sh', and run 'python strainxpress.py -fq <reads>.fq' (adding -fast for larger datasets), collecting stageb/final_contigs.fasta. Run the competing assemblers SPAdes, IDBA-UD, Minia3 and MEGAHIT on the same read sets, optionally applying StrainXpress's XC clustering as a pre-processing step. Assemble each of the five Gut Metagenome datasets individually and evaluate E. coli contigs against the StrainEst-derived ground truth. Finally, evaluate all assemblies with Quast, recording genome fraction, N50 and identity, and aggregate into a single CSV (real_data_benchmark.csv) with one row per assembler-dataset combination.

## Related tools

- **StrainXpress** (Strain-aware de novo metagenome assembler being benchmarked; run via 'python strainxpress.py -fq <reads>.fq') — https://github.com/HaploKit/StrainXpress
- **SPAdes** (Competing state-of-the-art assembler used as comparison baseline)
- **IDBA-UD** (Competing state-of-the-art assembler used as comparison baseline)
- **Minia3** (Competing state-of-the-art assembler used as comparison baseline)
- **MEGAHIT** (Competing state-of-the-art assembler used as comparison baseline)
- **Quast** (Assembly quality evaluation (genome fraction, N50, identity) of all assemblies)
- **minimap2** (Dependency of StrainXpress used in its overlap/alignment stage) — https://github.com/lh3/minimap2

## Examples

```
python strainxpress.py -fq all_reads.fq -fast
```

## Evaluation signals

- On Bmock12, StrainXpress achieves the highest genome fraction, exceeding all other assemblers by at least 3.7% and approaching 99.04% overall
- Quast tables report genome fraction, N50 and identity per assembler per sample; StrainXpress should rank first on genome fraction on real data
- The aggregated real_data_benchmark.csv contains one row for every assembler-dataset combination (assemblers × Bmock12 + NWCs + 5 Gut Metagenome samples)
- For Gut Metagenome, E. coli contigs match the StrainEst-derived ground truth of the 11 E. coli strains
- StrainXpress completes on large datasets with the -fast flag and outputs contigs in stageb/final_contigs.fasta

## Limitations

- No software version numbers are stated for StrainXpress or any compared tool, limiting exact reproducibility
- Short length of Illumina-type reads constrains contig length, preventing full-length strain-specific contigs
- High error rate of third-generation sequencing reads remains unresolved by this workflow
- The average coverage of some NWCs strains is low (minimum 7.46× for one Lactobacillus helveticus strain), which can challenge assemblers on poorly covered strains

## Evidence

- [results] On the real Bmock12 mock community, StrainXpress achieves the highest genome fraction, exceeding all other assemblers by at least 3.7% and approaching 99.04% overall: [evidence span withheld — anchor status: not_found]
- [intro] Assembler comparison set: "A non-exhaustive selection of state-of-the-art approaches encompasses IDBA-UD (6), SPAdes (7), Minia3 (8) and MEGAHIT (9)."
- [results] Quast-based evaluation: "Quantities referring to the criteria listed in the corresponding Tables were determined using Quast (40)"
- [results] XC pre-processing for other tools: "We considered the application of StrainXpress’s clustering procedure (referred to as ‘XC’ in Table 1) as a pre-processing step for the other methods"
- [results] Gut Metagenome per-sample assembly: "Reads were assembled for each of the five data sets individually"
- [readme] Conda installation commands: "conda install -c bioconda python=3.6 scipy pandas minimap2"
- [readme] Interleaved FASTQ input and -fast flag: "When a data set is big, we recommend to use the fast cluster method:"
- [abstract] StrainXpress claim on strain-specific reconstruction: "StrainXpress reconstructs strain-specific genomes from metagenomes that involve up to >1000 strains and proves to successfully deal with poorly covered strains."
