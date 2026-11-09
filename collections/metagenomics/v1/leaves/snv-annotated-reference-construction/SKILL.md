---
name: snv-annotated-reference-construction
description: Use when preparing a truth-set diploid reference (two haploid FASTAs)
  from a reference genome plus a VCF, as the required input for diploid Linked-Read
  simulation with LRTK-SIM.
when_to_use_negative:
- The VCF contains indels or structural variants that must be represented — this version
  of gen_fasta.py supports SNVs only
- The goal is real-data analysis rather than simulation — use the original reference
  genome directly instead of an SNV-modified template
- A haploid simulation is desired and no variant-aware template is needed (set Hap=1
  with a single FASTA instead)
edam_operation: http://edamontology.org/operation_0331
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: gen_fasta.py
  role: Auxiliary script bundled with LRTK-SIM that generates the two haploid FASTA
    files by inserting VCF variants into the reference genome
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: LRTK-SIM
  role: Downstream simulator that consumes the two haploid FASTAs via config lines
    2/3 (Path_Fastahap1/Path_Fastahap2) with Hap=2
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: python3
  role: Runtime for gen_fasta.py and LRTK-SIM; LRTK-SIM additionally requires numpy,
    multiprocessing, gzip, collections, os, sys
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:5d5d168b3e1fda68bb4dbfaf5e416cf1f9dfbf5d1ab7f33bbef082f4e8c91d5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhanglu295/LRTK-SIM
  tools:
  - LRTK-SIM
  - gen_fasta.py
  - multiprocessing
  - numpy
  - python3
  edam_operation: http://edamontology.org/operation_0331
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# snv-annotated-reference-construction

## Summary

Constructs a diploid simulation template by generating two haploid FASTA files in which known variants (NA12878 SNVs from a GIAB VCF) are inserted into a reference genome (hg38), providing the ground-truth haplotypes that LRTK-SIM shears and barcodes for synthetic 10x Chromium Linked-Read data.

## When to use

Use when preparing a truth-set diploid reference (two haploid FASTAs) from a reference genome plus a VCF, as the required input for diploid Linked-Read simulation with LRTK-SIM.

## When NOT to use

- The VCF contains indels or structural variants that must be represented — this version of gen_fasta.py supports SNVs only
- The goal is real-data analysis rather than simulation — use the original reference genome directly instead of an SNV-modified template
- A haploid simulation is desired and no variant-aware template is needed (set Hap=1 with a single FASTA instead)

## Inputs

- Variant VCF file (compressed or uncompressed), e.g. GIAB NA12878 SNV calls against hg38
- Reference genome FASTA (compressed or uncompressed), e.g. hg38
- Prefix string for the new reference sequences
- Output directory path

## Outputs

- Two haploid reference FASTA files (haplotype 1 and haplotype 2) with VCF SNVs inserted into the reference genome
- Derived LRTK-SIM config entries: Path_Fastahap1/Path_Fastahap2 (config lines 2/3) with Hap=2 (line 33)

## How to apply

Run gen_fasta.py with a variant VCF (-v) and a reference genome sequence (-r), a prefix for the new sequences (-p), and an output path (-o); the script writes two haploid FASTA files that differ only at the inserted variant positions. The VCF and reference may each be compressed or uncompressed, but this version supports SNVs only — indels and structural variants from the VCF are not inserted. Set Hap=2 and supply the two haploid FASTA paths (Path_Fastahap1, Path_Fastahap2) in the LRTK-SIM config lines 2/3 so the simulator models a diploid genome; for a haploid run, remove Path_Fastahap2 and set Hap=1. Rationale: the 2:1 het:hom NA12878 SNV pattern at ~1 SNV/kb in low-confidence regions gives LRTK-SIM a realistic diploid template whose known phase serves as the gold standard for evaluating assembly accuracy.

## Related tools

- **gen_fasta.py** (Auxiliary script bundled with LRTK-SIM that generates the two haploid FASTA files by inserting VCF variants into the reference genome) — https://github.com/zhanglu295/LRTK-SIM
- **LRTK-SIM** (Downstream simulator that consumes the two haploid FASTAs via config lines 2/3 (Path_Fastahap1/Path_Fastahap2) with Hap=2) — https://github.com/zhanglu295/LRTK-SIM
- **python3** (Runtime for gen_fasta.py and LRTK-SIM; LRTK-SIM additionally requires numpy, multiprocessing, gzip, collections, os, sys)

## Examples

```
python gen_fasta.py -v sample.vcf -r ref.fasta -p newref -o ./work
```

## Evaluation signals

- Two distinct haploid FASTA files are produced with the requested prefix in the output directory
- The two haplotypes differ only at SNV positions from the input VCF (SNV-only, per version limitation)
- Config files reference both haplotype paths at lines 2/3 and set Hap=2 at line 33 for diploid simulation
- Downstream LRTK-SIM run completes and writes lib_configX/ folders containing X_S1_L001_R1_001.fastq.gz and X_S1_L001_R2_001.fastq.gz accepted by Long Ranger and Supernova

## Limitations

- Only SNVs are supported in this version of gen_fasta.py; indels and structural variants from the VCF are not inserted into the haploid sequences
- The two haplotypes differ only at SNVs, so the simulated diploid template does not model structural variation between haplotypes
- Both the VCF and reference must be readable by the script (compressed or uncompressed are allowed)

## Evidence

- [methods] gen_fasta.py can generate two haploid fasta files with variants from vcf and reference genome sequence.: "`gen_fasta.py` can generate two haploid fasta files with variants from vcf and reference genome sequence."
- [readme] The diploid reference sequences can be generated by gen_fasta.py that inserting variants to the reference genome (only SNVs for this version).: "The diploid reference sequences can be generated by `gen_fasta.py` that inserting variants to the reference genome (only SNVs for this version)."
- [readme] usage: python gen_fasta.py -v sample.vcf -r ref.fasta -p newref -o ./work: "`usage: python gen_fasta.py -v sample.vcf -r ref.fasta -p newref -o ./work`"
