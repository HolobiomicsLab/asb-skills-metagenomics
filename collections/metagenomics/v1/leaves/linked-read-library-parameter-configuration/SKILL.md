---
name: linked-read-library-parameter-configuration
description: Use when planning or simulating 10x Chromium Linked-Read libraries and
  you need to set physical fragment coverage (CF), per-fragment read coverage (CR),
  fragment length (Mu_F), and droplet fragment counts (N_FP) for Supernova2 assembly
  testing.
when_to_use_negative:
- Input reference contains structural variants or indels — gen_fasta.py supports only
  SNVs in this version
- You need real sequencing data rather than simulation — the article supplements simulations
  with real high-depth data from NA12878 and NA24385 (PRJNA527321)
- Haploid simulation is desired but Hap is left at 2 — you must remove Path_Fastahap2
  and set Hap=1 (line 33)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: LRTK-SIM
  role: Simulates 10x Chromium Linked-Read libraries from configured parameters, emitting
    barcoded Illumina paired-end FASTQs
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: gen_fasta.py
  role: Auxiliary LRTK-SIM program that generates two haploid FASTAs by inserting
    VCF SNVs into the reference genome
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: Supernova2
  role: Downstream de novo diploid assembler used to evaluate assemblies produced
    from the simulated libraries
- name: Long Ranger
  role: 10x official pipeline that accepts the simulated FASTQs as an acceptance check
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:3bb20fb55dbb8489565995b7a59cf5553d70a6d5ee29620a05988b52d4fbb710
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhanglu295/LRTK-SIM
  tools:
  - LRTK-SIM
  - Long Ranger
  - Supernova2
  - gen_fasta.py
  - multiprocessing
  - numpy
  - python3
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# linked-read-library-parameter-configuration

## Summary

Configure and simulate 10x Chromium Linked-Read libraries by setting CF, CR, Mu_F, N_FP, insert-size, and read parameters in LRTK-SIM config files, generating realistic synthetic diploid data. Used to constrain the 10x library-parameter space (e.g., optimal CF 332x–823x, WμFL ∼50–150 kb) before costly wet-lab library preparation.

## When to use

Use when planning or simulating 10x Chromium Linked-Read libraries and you need to set physical fragment coverage (CF), per-fragment read coverage (CR), fragment length (Mu_F), and droplet fragment counts (N_FP) for Supernova2 assembly testing.

## When NOT to use

- Input reference contains structural variants or indels — gen_fasta.py supports only SNVs in this version
- You need real sequencing data rather than simulation — the article supplements simulations with real high-depth data from NA12878 and NA24385 (PRJNA527321)
- Haploid simulation is desired but Hap is left at 2 — you must remove Path_Fastahap2 and set Hap=1 (line 33)

## Inputs

- hg38 reference genome FASTA (compressed or uncompressed)
- GIAB NA12878 SNV VCF (compressed or uncompressed) for gen_fasta.py
- LRTK-SIM configuration file (e.g., config1.txt) with Path_Fastahap1/hap2, CF, CR, N_FP, Mu_F, SR, Error_rate, Mu_IS, Std_IS, Hap
- Optional: metagenomics abundance flat file (sequence name, abundance summing to 1)

## Outputs

- Per-library folder lib_configX containing gzipped paired-end FASTQ files: configX_S1_L001_R1_001.fastq.gz (forward) and configX_S1_L001_R2_001.fastq.gz (reverse)
- Synthetic 10x Linked-Read data accepted by Long Ranger and Supernova2 pipelines

## How to apply

Build a diploid template by inserting GIAB NA12878 SNVs from a VCF into hg38 with gen_fasta.py (only SNVs supported), producing two haploid FASTAs. Fill a config file (e.g., config1.txt) with: Path_Fastahap1/hap2 (lines 2–3), processors (line 5), CF (line 7), CR (line 9), N_FP (line 11), Mu_F in Kb (line 13), SR short-read length (line 15), Error_rate (line 21), Mu_IS/Std_IS insert-size mean and SD (lines 27/29), and Hap=2 for diploid simulation (line 33). Run 'python LRTK-SIM.py ./diploid_config'; multiple libraries simulate simultaneously. When choosing values, keep total sequencing coverage C near the 10x-recommended 56x while holding CF between ~335x and 823x (no larger than 1,000x), and target a length-weighted fragment length of ∼50–150 kb, since extremely long fragments degraded Supernova2 assembly quality in this study.

## Related tools

- **LRTK-SIM** (Simulates 10x Chromium Linked-Read libraries from configured parameters, emitting barcoded Illumina paired-end FASTQs) — https://github.com/zhanglu295/LRTK-SIM
- **gen_fasta.py** (Auxiliary LRTK-SIM program that generates two haploid FASTAs by inserting VCF SNVs into the reference genome) — https://github.com/zhanglu295/LRTK-SIM
- **Supernova2** (Downstream de novo diploid assembler used to evaluate assemblies produced from the simulated libraries)
- **Long Ranger** (10x official pipeline that accepts the simulated FASTQs as an acceptance check)

## Examples

```
python gen_fasta.py -v sample.vcf -r ref.fasta -p newref -o ./work; python LRTK-SIM.py ./diploid_config
```

## Evaluation signals

- Each simulated library folder (lib_config1, lib_config2, ...) contains both configX_S1_L001_R1_001.fastq.gz and configX_S1_L001_R2_001.fastq.gz
- Simulated FASTQs are accepted by 10x official pipelines Long Ranger and Supernova2
- Config file lines match the documented layout: CF line 7, CR line 9, N_FP line 11, Mu_F line 13, SR line 15, Error_rate line 21, Mu_IS line 27, Std_IS line 29, Hap line 33
- gen_fasta.py produced exactly two haploid FASTAs for diploid simulation (Hap=2), with only SNV variants inserted
- Chosen parameter values fall within the empirically optimal ranges: CF ≈335–823x (never >1,000x at C=56x) and WμFL ∼50–150 kb

## Limitations

- gen_fasta.py supports only SNVs; indels and structural variants from the VCF are not inserted
- Assembly quality worsens if CF exceeds 1,000x for a given C, and including more extremely long fragments was detrimental (proposed cause: loss of barcode specificity across repeats, unverified)
- Interpretation of the fragment-length effect is limited because no alternative assembler to Supernova2 was available for comparison
- Simultaneous simulation of multiple libraries requires one config file per library (config1.txt, config2.txt, ...) in the same folder

## Evidence

- [methods] python LRTK-SIM.py <path to configuration files> Multiple libraries are allowed to simulated simutaneously: "`python LRTK-SIM.py <path to configuration files>`

Multiple libraries are allowed to simulated simutaneously"
- [readme] gen_fasta.py can generate two haploid fasta files with variants from vcf and reference genome sequence.: "`gen_fasta.py` can generate two haploid fasta files with variants from vcf and reference genome sequence."
- [abstract] The optimal physical coverage was between 332× and 823× and assembly quality worsened if it increased to >1,000× for a given C.: "The optimal physical coverage was between 332× and 823× and assembly quality worsened if it increased to >1,000× for a given C."
- [abstract] The optimal length-weighted fragment length (W${\mu _{FL}}$) was ∼50–150 kb.: [evidence span withheld — anchor status: not_found]
- [readme] The simulated data have been tested and accepted by 10X official pipelines Long Ranger and Supernova.: "The simulated data have been tested and accepted by 10X official pipelines [Long Ranger]"
