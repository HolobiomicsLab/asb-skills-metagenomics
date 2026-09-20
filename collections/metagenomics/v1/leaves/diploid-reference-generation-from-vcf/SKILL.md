---
name: diploid-reference-generation-from-vcf
description: Use when simulating diploid 10x Linked-Read libraries (LRTK-SIM Hap=2)
  that require two haploid reference sequences derived from a reference genome plus
  known variants, or when a gold standard for diploid assembly accuracy is needed.
when_to_use_negative:
- Your VCF contains structural variants or indels that must be represented — gen_fasta.py
  in this version supports SNVs only
- You need haploid simulation only — in that case a single reference FASTA suffices
  and LRTK-SIM should be run with Hap=1 and no Path_Fastahap2 entry
- You need a diploid reference for real (non-simulated) data analysis rather than
  for simulation ground truth
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: gen_fasta.py
  role: Auxiliary program of LRTK-SIM that generates two haploid FASTA files by inserting
    SNVs from a VCF into the reference genome sequence
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: LRTK-SIM
  role: 'Downstream consumer: takes the two haploid FASTAs (Path_Fastahap1/Path_Fastahap2)
    to simulate diploid 10x Chromium Linked-Read libraries'
  repo: https://github.com/zhanglu295/LRTK-SIM
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:ad0f0130b0cba7cb20c345a7d2b477a91aa16c56731d5348d4938c09c802e7e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhanglu295/LRTK-SIM
  tools:
  - LRTK-SIM
  - Python
  - Supernova2
  - gen_fasta.py
  - multiprocessing
  - numpy
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# diploid-reference-generation-from-vcf

## Summary

Generate two haploid reference FASTA files by inserting SNVs from a VCF into the hg38 reference genome using gen_fasta.py, creating a diploid ground truth for simulating Linked-Read data with LRTK-SIM and for validating diploid assemblies.

## When to use

Use when simulating diploid 10x Linked-Read libraries (LRTK-SIM Hap=2) that require two haploid reference sequences derived from a reference genome plus known variants, or when a gold standard for diploid assembly accuracy is needed.

## When NOT to use

- Your VCF contains structural variants or indels that must be represented — gen_fasta.py in this version supports SNVs only
- You need haploid simulation only — in that case a single reference FASTA suffices and LRTK-SIM should be run with Hap=1 and no Path_Fastahap2 entry
- You need a diploid reference for real (non-simulated) data analysis rather than for simulation ground truth

## Inputs

- Variant call file (VCF), compressed or uncompressed, containing SNVs
- Reference genome FASTA (e.g. hg38), compressed or uncompressed
- Output prefix string (-p, e.g. newref)
- Output directory path (-o, e.g. ./work)

## Outputs

- Two haploid reference FASTA files (hap1 and hap2) with SNVs inserted from the VCF
- Downstream inputs for LRTK-SIM diploid simulation via Path_Fastahap1/Path_Fastahap2 config entries

## How to apply

Run `python gen_fasta.py -v sample.vcf -r ref.fasta -p newref -o ./work`, supplying a compressed or uncompressed VCF of variants, the compressed or uncompressed reference FASTA (hg38 in this study), a prefix for the new sequences, and an output path. The tool splits the variant calls across two haploid reference FASTA files so that the simulated genome is diploid. Note this version supports SNVs only — it does not insert structural variants into the reference. The two output haploid FASTAs are then pointed to via `Path_Fastahap1` (line2) and `Path_Fastahap2` (line3) in LRTK-SIM configuration files with `Hap=2` (line33) so the simulator draws fragments from both haplotypes, matching the workflow used to reproduce Supernova2 contiguity trends across total coverage C = 16x to 78x.

## Related tools

- **gen_fasta.py** (Auxiliary program of LRTK-SIM that generates two haploid FASTA files by inserting SNVs from a VCF into the reference genome sequence) — https://github.com/zhanglu295/LRTK-SIM
- **LRTK-SIM** (Downstream consumer: takes the two haploid FASTAs (Path_Fastahap1/Path_Fastahap2) to simulate diploid 10x Chromium Linked-Read libraries) — https://github.com/zhanglu295/LRTK-SIM

## Examples

```
python gen_fasta.py -v sample.vcf -r ref.fasta -p newref -o ./work
```

## Evaluation signals

- Two distinct haploid FASTA files are produced in the output directory, distinguishable by the supplied prefix
- Haplotypes differ only at SNV positions from the VCF; no structural variants are present (SNV-only version)
- The FASTAs are accepted by LRTK-SIM: config lines 2 and 3 (Path_Fastahap1/Path_Fastahap2) point to them and Hap=2 at line 33 runs a diploid simulation
- Simulated libraries built from these references are accepted by the 10x official pipelines Long Ranger and Supernova, per the LRTK-SIM README

## Limitations

- gen_fasta.py in this version supports SNVs only; other variant classes are not inserted into the haploid references
- The hg38 reference already contains some structural variants, which is separate from what the tool can insert from the VCF

## Evidence

- [readme] gen_fasta.py can generate two haploid fasta files with variants from vcf and reference genome sequence.: "`gen_fasta.py` can generate two haploid fasta files with variants from vcf and reference genome sequence."
- [readme] The diploid reference sequences can be generated by gen_fasta.py that inserting variants to the reference genome (only SNVs for this version).: "The diploid reference sequences can be generated by `gen_fasta.py` that inserting variants to the reference genome (only SNVs for this version)."
- [supplementary] Generate the two haploid diploid reference FASTAs by running gen_fasta.py inserting SNVs from the VCF into the hg38 reference: [evidence span withheld — anchor status: not_found]
