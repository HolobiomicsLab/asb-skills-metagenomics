---
name: diploid-genome-assembly
description: Use when the goal is to assemble a diploid genome from 10x Chromium Linked-Read
  libraries, or to test how source-genome features (repeats, heterozygous variation,
  sequencing error) or library parameters (CF, CR, mu_FL) affect Supernova2 diploid
  assembly quality.
when_to_use_negative:
- The input is already an assembled genome or a non-Linked-Read technology (e.g.,
  PacBio/ONT long reads or standard short-read data) — the skill is specific to 10x
  Chromium Linked-Read data and Supernova2
- The organism lacks a high-quality reference genome, since NA50-based accuracy comparison
  depends on an existing gold standard
- Variant insertion is needed beyond SNVs — the gen_fasta.py version described only
  inserts SNVs from the VCF
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: LRTK-SIM
  role: Simulates realistic 10x Linked-Read data from one or two haploid fasta references,
    parameterized by CF, CR, Mu_F, and Error_rate; outputs paired gzipped fastq files
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: gen_fasta.py
  role: Auxiliary program in LRTK-SIM that generates two haploid fasta files with
    variants from a VCF and reference genome (SNVs only) for diploid simulation
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: Supernova2
  role: De novo diploid genome assembler used to assemble the simulated and real 10x
    Linked-Read libraries, requesting megabubbles output
- name: Evaluate_diploid_assembly
  role: Provides notebook-based code (Evaluate_assembly_performance.ipynb) to evaluate
    and compare diploid assemblies of different parameters
  repo: https://github.com/zhanglu295/Evaluate_diploid_assembly
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:912f86046052c44bd5a5159f0c90eb38a95fcb29ac7bdc55fdf5ff880a12615a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhanglu295/LRTK-SIM
  tools:
  - Evaluate_diploid_assembly
  - LRTK-SIM
  - Supernova2
  - gen_fasta.py
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# diploid-genome-assembly

## Summary

Assemble and evaluate diploid (haplotype-resolved) human genomes from 10x Linked-Read data using LRTK-SIM for simulation, gen_fasta.py for diploid reference construction, and Supernova2 for de novo assembly, with contiguity (N50/NA50) and accuracy metrics judged against a gold standard.

## When to use

Use when the goal is to assemble a diploid genome from 10x Chromium Linked-Read libraries, or to test how source-genome features (repeats, heterozygous variation, sequencing error) or library parameters (CF, CR, mu_FL) affect Supernova2 diploid assembly quality.

## When NOT to use

- The input is already an assembled genome or a non-Linked-Read technology (e.g., PacBio/ONT long reads or standard short-read data) — the skill is specific to 10x Chromium Linked-Read data and Supernova2
- The organism lacks a high-quality reference genome, since NA50-based accuracy comparison depends on an existing gold standard
- Variant insertion is needed beyond SNVs — the gen_fasta.py version described only inserts SNVs from the VCF

## Inputs

- Reference genome sequence (e.g., hg38 human reference fasta)
- VCF of known heterozygous variants (compressed or uncompressed) for gen_fasta.py
- Chromosome or genome fasta source sequences (unmodified, repeat-masked, variation-removed, or error-introduced)
- LRTK-SIM configuration files specifying CF, CR, N_FP, Mu_F, SR, Error_rate, Mu_IS, Std_IS, Hap
- Real 10x Chromium Linked-Read fastq libraries (e.g., NA12878, NA24385)

## Outputs

- Two gzipped fastq files per simulated library (X_S1_L001_R1_001.fastq.gz, X_S1_L001_R2_001.fastq.gz) in lib_X folders
- Supernova2 de novo diploid assembly (megabubbles output): contigs, scaffolds, and phase blocks
- Assembly comparison report: contig and scaffold N50/NA50 and accuracy metrics across parameter combinations

## How to apply

Generate two haploid fasta files with variants from a VCF using gen_fasta.py (SNVs only), setting Path_Fastahap1/Path_Fastahap2 and Hap=2 in the LRTK-SIM config files. Simulate libraries with LRTK-SIM (python LRTK-SIM.py <config folder>), sweeping CF (physical fragment coverage), CR (read coverage per fragment), and Mu_F (average fragment length in Kb); collect the paired gzipped fastq outputs (X_S1_L001_R1_001.fastq.gz / X_S1_L001_R2_001.fastq.gz) from each lib_X folder. Assemble each library with Supernova2 requesting the megabubbles output, then compare contig/scaffold N50 and NA50 plus accuracy metrics across conditions. Accept results when parameter-guided changes (e.g., repeat-masked source) improve contiguity with no accuracy loss; note that optimal physical coverage is 332x–823x and never above 1,000x at C = 56x, and WμFL of ~50–150 kb.

## Related tools

- **LRTK-SIM** (Simulates realistic 10x Linked-Read data from one or two haploid fasta references, parameterized by CF, CR, Mu_F, and Error_rate; outputs paired gzipped fastq files) — https://github.com/zhanglu295/LRTK-SIM
- **gen_fasta.py** (Auxiliary program in LRTK-SIM that generates two haploid fasta files with variants from a VCF and reference genome (SNVs only) for diploid simulation) — https://github.com/zhanglu295/LRTK-SIM
- **Supernova2** (De novo diploid genome assembler used to assemble the simulated and real 10x Linked-Read libraries, requesting megabubbles output)
- **Evaluate_diploid_assembly** (Provides notebook-based code (Evaluate_assembly_performance.ipynb) to evaluate and compare diploid assemblies of different parameters) — https://github.com/zhanglu295/Evaluate_diploid_assembly

## Examples

```
python gen_fasta.py -v sample.vcf -r ref.fasta -p newref -o ./work && python LRTK-SIM.py ./diploid_config
```

## Evaluation signals

- Repeat-masked source genome yields improved contig contiguity with no loss of assembly accuracy relative to the unmodified source
- Variation-removed source lengthens scaffolds only in the CR > 0.8x conditions
- 1%-uniform-error (Error_rate = 0.01) source shows no discernible effect relative to the unmodified source
- Simulated fastq files are accepted by 10x official pipelines (Long Ranger and Supernova)
- Physical coverage CF falls within the optimal 332x–823x range (never >1,000x) and WμFL within ~50–150 kb for broadly optimal assemblies

## Limitations

- Only SNVs can be inserted by gen_fasta.py; other variant classes are not supported in this version
- Assembly quality worsens if physical coverage increases above 1,000x for a given total coverage C; extremely long fragments are detrimental to assembly quality
- No alternative assembler to Supernova2 was available for comparison, limiting interpretation of the fragment-length effect; the harm from extremely long fragments (loss of barcode specificity) is a proposed but unverified explanation
- NA50-based accuracy assessment requires a high-quality reference genome; otherwise N50 alone is misleading

## Evidence

- [abstract] We also developed the simulator LRTK-SIM to follow the workflow of 10x data generation and produce realistic simulated Linked-Read data sets.: "We also developed the simulator LRTK-SIM to follow the workflow of 10x data generation and produce realistic simulated Linked-Read data sets."
- [readme] gen_fasta.py usage: "`gen_fasta.py` can generate two haploid fasta files with variants from vcf and reference genome sequence."
- [readme] LRTK-SIM basic usage: "python LRTK-SIM.py <path to configuration files>"
- [discussion] Optimal physical coverage: [evidence span withheld — anchor status: not_found]
- [readme] Assembly output format: "Two gzipped fastq files are generated and started by X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz to represent forward and reverse reads."
