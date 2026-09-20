---
name: variant-insertion-reference-generation
description: Use when simulating diploid linked-read data from a reference genome
  plus a variant call set (VCF), where two haploid FASTA inputs with inserted SNVs
  are needed as LRTK-SIM source sequences.
when_to_use_negative:
- When structural variants or indels need to be inserted — gen_fasta.py in this version
  inserts only SNVs into the reference genome
- For haploid simulation from a single unmodified reference — remove Path_Fastahap2
  and set Hap=1 instead of generating variant FASTAs
- When real 10x Linked-Read data are available rather than simulated data requiring
  a known gold standard
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: LRTK-SIM
  role: Hosts the auxiliary gen_fasta.py script and consumes the generated haploid
    FASTA files as diploid source sequences for linked-read simulation
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: gen_fasta.py
  role: Generates two haploid FASTA files by inserting variants from the VCF into
    the reference genome sequence (SNVs only)
  repo: https://github.com/zhanglu295/LRTK-SIM
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:616173e6a0c9bc8bc05db2920049fd633f35c5c3ca1dff5a58a0aff884c6fe4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhanglu295/LRTK-SIM
  tools:
  - LRTK-SIM
  - Supernova2
  - gen_fasta.py
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# variant-insertion-reference-generation

## Summary

Generate two haploid reference FASTA files carrying known variants from a VCF using LRTK-SIM's auxiliary gen_fasta.py, producing a controlled diploid gold standard for simulating 10x Linked-Read data (SNVs only).

## When to use

Use when simulating diploid linked-read data from a reference genome plus a variant call set (VCF), where two haploid FASTA inputs with inserted SNVs are needed as LRTK-SIM source sequences.

## When NOT to use

- When structural variants or indels need to be inserted — gen_fasta.py in this version inserts only SNVs into the reference genome
- For haploid simulation from a single unmodified reference — remove Path_Fastahap2 and set Hap=1 instead of generating variant FASTAs
- When real 10x Linked-Read data are available rather than simulated data requiring a known gold standard

## Inputs

- Compressed or uncompressed VCF file of sample variants (e.g. GIAB benchmark calls)
- Compressed or uncompressed reference genome sequence in FASTA format (e.g. hg38 human reference)

## Outputs

- Two haploid FASTA files with SNVs inserted, one per haplotype, suitable as LRTK-SIM Path_Fastahap1/Path_Fastahap2 inputs

## How to apply

Run gen_fasta.py with a VCF and reference genome sequence to generate two haploid FASTA files with variants inserted; the current version supports only SNVs. Point LRTK-SIM's config file lines Path_Fastahap1 and Path_Fastahap2 to the two haploid FASTA outputs and set Hap=2 for diploid simulation (a single FASTA with Hap=1 gives haploid simulation). The resulting haploid sequences serve as the known gold standard against which Supernova2 diploid assemblies (contig/scaffold N50/NA50) are evaluated. For chromosome 19 source-genome perturbation experiments, one of the four prepared sources (unmodified, repeat-masked, variation-removed, 1% uniform sequencing error) is combined with these haploid variant sequences.

## Related tools

- **LRTK-SIM** (Hosts the auxiliary gen_fasta.py script and consumes the generated haploid FASTA files as diploid source sequences for linked-read simulation) — https://github.com/zhanglu295/LRTK-SIM
- **gen_fasta.py** (Generates two haploid FASTA files by inserting variants from the VCF into the reference genome sequence (SNVs only)) — https://github.com/zhanglu295/LRTK-SIM

## Examples

```
python gen_fasta.py -v sample.vcf -r ref.fasta -p newref -o ./work
```

## Evaluation signals

- The two output FASTA files each contain the full diploid set of SNVs from the input VCF inserted at the correct reference coordinates (SNVs only in this version)
- Setting Path_Fastahap1/Path_Fastahap2 to the outputs and Hap=2 in the LRTK-SIM config runs without errors and produces paired gzipped fastq files (X_S1_L001_R1_001.fastq.gz / X_S1_L001_R2_001.fastq.gz)
- Simulated reads from the two haplotypes reflect known variant positions, enabling downstream NA50/N50 accuracy comparison against the gold standard
- Output folder naming follows the expected lib_X convention next to the configuration files

## Limitations

- gen_fasta.py in this version supports only SNVs — indels and structural variants cannot be inserted
- The variation-removed source showed scaffold-lengthening benefits only in CR > 0.8x conditions, so variant-removal effects are coverage-dependent
- Interpreting diploid assembly accuracy from these simulated gold standards relies on NA50, which is only computable because a high-quality human reference genome exists

## Evidence

- [readme] gen_fasta.py can generate two haploid fasta files with variants from vcf and reference genome sequence.: "`gen_fasta.py` can generate two haploid fasta files with variants from vcf and reference genome sequence."
- [readme] Path_Fastahap1 and Path_Fastahap2, the two haploid reference serquences. LRTK-SIM allows one or two fasta files to perform haploid and diploid simulation.: "`line2` and `line3`: `Path_Fastahap1` and `Path_Fastahap2`, the two haploid reference serquences. LRTK-SIM allows one or two fasta files to perform haploid and diploid simulation."
- [readme] The diploid reference sequences can be generated by gen_fasta.py that inserting variants to the reference genome (only SNVs for this version).: "The diploid reference sequences can be generated by `gen_fasta.py` that inserting variants to the reference genome (only SNVs for this version)."
- [other] removing interspersed repeat sequences from the source genome produced better contigs with no loss of assembly accuracy: [evidence span withheld — anchor status: not_found]
- [discussion] which we were able to compare to NA50 because there exists a high-quality human reference genome: "which we were able to compare to NA50 because there exists a high-quality human reference genome"
