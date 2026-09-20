---
name: diploid-reference-variant-injection
description: Use when preparing diploid in silico reference genomes for Linked-Read
  read simulation, e.g. to reproduce assembly-accuracy (NA50) benchmarks from real
  NA12878/NA24385-style data.
when_to_use_negative:
- Your study requires haploid simulation only — remove Path_Fastahap2 and set Hap=1
  instead of building two haploid references
- Your variant set contains structural variants or indels — gen_fasta.py inserts only
  SNVs in this version, so SV-bearing references cannot be produced this way
- Input is already simulated or real 10x FASTQ data and no new in silico libraries
  are needed
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: gen_fasta.py
  role: Auxiliary script that generates two haploid FASTA files by inserting VCF SNVs
    into the reference genome, producing the diploid reference for simulation
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: LRTK-SIM
  role: Simulator that consumes the two haploid FASTAs via config files to produce
    realistic simulated 10x Linked-Read FASTQ data sets
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: Supernova2
  role: Downstream assembler used to produce de novo diploid assemblies from the simulated
    reads for quality evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:e8a59a9a2b79196b097de1771928e87b9363ee69d06b8ae8228aeb1e349bd5e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhanglu295/LRTK-SIM
  tools:
  - LRTK-SIM
  - Supernova2
  - gen_fasta.py
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# diploid-reference-variant-injection

## Summary

Generate two haploid reference FASTA files by injecting sample SNVs from a VCF into the hg38 reference genome with gen_fasta.py, creating the diploid inputs required for realistic 10x Linked-Read simulation with LRTK-SIM.

## When to use

Use when preparing diploid in silico reference genomes for Linked-Read read simulation, e.g. to reproduce assembly-accuracy (NA50) benchmarks from real NA12878/NA24385-style data.

## When NOT to use

- Your study requires haploid simulation only — remove Path_Fastahap2 and set Hap=1 instead of building two haploid references
- Your variant set contains structural variants or indels — gen_fasta.py inserts only SNVs in this version, so SV-bearing references cannot be produced this way
- Input is already simulated or real 10x FASTQ data and no new in silico libraries are needed

## Inputs

- Sample VCF file (compressed or uncompressed) with SNV variants
- Reference genome FASTA (e.g. hg38 human reference sequence, compressed or uncompressed)
- LRTK-SIM configuration files specifying Path_Fastahap1/Path_Fastahap2, processors, CF, CR, Mu_F, SR, Error_rate, Mu_IS, Std_IS, and Hap=2

## Outputs

- Two haploid reference FASTA files with injected SNVs
- Per-library output folders (lib_configX) containing paired gzipped FASTQ files X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz in 10x naming format

## How to apply

Run gen_fasta.py with a sample VCF (-v) and the reference genome FASTA (-r), setting a prefix (-p) and output path (-o), to produce two haploid FASTA files carrying the sample's variants (SNVs only in this version). Write one configuration file per simulated library (e.g. config1.txt, config2.txt in a diploid_config folder) pointing Path_Fastahap1/Path_Fastahap2 at the two haploid FASTAs, set Hap=2 for diploid simulation, and vary coverage parameters CF and CR (with C = CR × CF) plus Mu_F, SR, Error_rate, Mu_IS, and Std_IS. Launch LRTK-SIM on the configuration folder so multiple libraries are simulated simultaneously. Use the resulting simulated reads as a gold standard to constrain the 10x parameter space before generating real high-depth libraries from NA12878 or NA24385.

## Related tools

- **gen_fasta.py** (Auxiliary script that generates two haploid FASTA files by inserting VCF SNVs into the reference genome, producing the diploid reference for simulation) — https://github.com/zhanglu295/LRTK-SIM
- **LRTK-SIM** (Simulator that consumes the two haploid FASTAs via config files to produce realistic simulated 10x Linked-Read FASTQ data sets) — https://github.com/zhanglu295/LRTK-SIM
- **Supernova2** (Downstream assembler used to produce de novo diploid assemblies from the simulated reads for quality evaluation)

## Examples

```
python gen_fasta.py -v sample.vcf -r ref.fasta -p newref -o ./work
```

## Evaluation signals

- Each library folder (lib_configX) contains both gzipped FASTQ files X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz representing forward and reverse reads
- The simulated FASTQ files are accepted by the 10x official pipelines Long Ranger and Supernova without format errors
- The config files correctly reference the two haploid FASTAs (Path_Fastahap1/Path_Fastahap2) and set Hap=2 for diploid simulation
- Coverage parameters in the config follow the intended C = CR × CF design (e.g. varying C from 48x to 67x), and achieved assemblies show expected N50/NA50 trends when assembled with Supernova2

## Limitations

- gen_fasta.py in this version injects only SNVs from the VCF — indels and structural variants are not captured in the simulated diploid references
- NA50-based accuracy evaluation is only possible because a high-quality human reference genome exists; for other organisms N50 alone is misleading
- The harm from certain parameter choices (e.g. extremely long fragments) reflects Supernova2's specific implementation, as no alternative assembler was available for comparison

## Evidence

- [readme] gen_fasta.py can generate two haploid fasta files with variants from vcf and reference genome sequence.: "`gen_fasta.py` can generate two haploid fasta files with variants from vcf and reference genome sequence."
- [readme] LRTK-SIM allows one or two fasta files to perform haploid and diploid simulation.: "LRTK-SIM allows one or two fasta files to perform haploid and diploid simulation. The diploid reference sequences can be generated by `gen_fasta.py` that inserting variants to the reference genome"
- [readme] The simulated data have been tested and accepted by 10X official pipelines Long Ranger and Supernova.: "The simulated data have been tested and accepted by 10X official pipelines [Long Ranger](https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger) and"
- [intro] We finally assessed the quality of the assemblies using standard metrics of contiguity and accuracy, facilitated by the existence of a gold standard (in the case of simulations): "We finally assessed the quality of the assemblies using standard metrics of contiguity and accuracy, facilitated by the existence of a gold standard (in the case of simulations)"
