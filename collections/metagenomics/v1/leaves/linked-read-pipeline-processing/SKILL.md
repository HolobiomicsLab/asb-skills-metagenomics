---
name: linked-read-pipeline-processing
description: Use when input is 10x Chromium Linked-Read data (real or simulated) for
  NA12878/NA24385-style genomes and the goal is SNV/SV calling from reference-based
  pipelines or Supernova2 de novo assemblies, benchmarked against GIAB truth sets.
when_to_use_negative:
- Input is PacBio or Oxford Nanopore long-read data for direct assembly rather than
  barcoded 10x Linked-Read libraries
- The genome of interest has no high-quality reference or GIAB benchmark sets, making
  vcfeval/truvari concordance benchmarking impossible
- Only haploid simulation is needed and diploid variant insertion via gen_fasta.py
  (SNVs only in this version) would be a mismatch
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Long Ranger
  role: Reference-aligned 10x Linked-Read pipeline producing SNV and deletion calls
    from Chromium libraries
  repo: https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger
- name: Supernova2
  role: De novo diploid whole-genome assembler for the 10x data; the assembly is also
    the substrate for assembly-based variant calling
  repo: https://support.10xgenomics.com/de-novo-assembly/software/overview/welcome
- name: LRTK-SIM
  role: Simulator that follows the 10x data-generation workflow to produce realistic
    Linked-Read FASTQs for parameter-space exploration
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: gen_fasta.py
  role: Auxiliary LRTK-SIM script that builds two haploid FASTA inputs by inserting
    SNVs from a VCF into the reference genome
  repo: https://github.com/zhanglu295/LRTK-SIM
- name: BWA-MEM
  role: Aligns NA12878/NA24385 Illumina short reads to the hg38 reference prior to
    FreeBayes/SvABA calling
- name: FreeBayes
  role: Calls SNVs from the BWA-MEM Illumina alignments for the reference-based comparison
    arm
- name: SvABA
  role: Calls SVs from the same BWA-MEM Illumina alignments
- name: RTG vcfeval
  role: Genotype-aware benchmarking of SNV call sets against GIAB truth sets
- name: truvari
  role: Benchmarks SV call sets against GIAB Tier 1 SV regions
- name: VaPoR
  role: Validates assembly-derived SV calls by aligning them to PacBio CCS long reads
- name: Evaluate_diploid_assembly
  role: Notebook-based code for comparing diploid assemblies of different parameters
  repo: https://github.com/zhanglu295/Evaluate_diploid_assembly
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:bc0f5122200ed3e293bb04eb3a586bc7091f05e00bc059bcfdad46b808db7ac5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA-MEM
  - Evaluate_diploid_assembly
  - FreeBayes
  - LRTK-SIM
  - Long Ranger
  - PacBio
  - RTG vcfeval
  - Supernova2
  - SvABA
  - VaPoR
  - gen_fasta.py
  - truvari
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# linked-read-pipeline-processing

## Summary

End-to-end processing of 10x Chromium Linked-Read data: simulating or sequencing libraries, running the Long Ranger pipeline and Supernova2 de novo assembly, and benchmarking the resulting variant calls against GIAB standards.

## When to use

Use when input is 10x Chromium Linked-Read data (real or simulated) for NA12878/NA24385-style genomes and the goal is SNV/SV calling from reference-based pipelines or Supernova2 de novo assemblies, benchmarked against GIAB truth sets.

## When NOT to use

- Input is PacBio or Oxford Nanopore long-read data for direct assembly rather than barcoded 10x Linked-Read libraries
- The genome of interest has no high-quality reference or GIAB benchmark sets, making vcfeval/truvari concordance benchmarking impossible
- Only haploid simulation is needed and diploid variant insertion via gen_fasta.py (SNVs only in this version) would be a mismatch

## Inputs

- 10x Chromium Linked-Read FASTQ files (R1/R2 pairs, e.g. X_S1_L001_R1_001.fastq.gz)
- Illumina short-read FASTQ files from NA12878 and NA24385
- hg38 human reference sequence (FASTA)
- GIAB benchmark variant call sets (VCF) and Tier 1 SV regions (BED)
- PacBio CCS long reads for SV validation
- LRTK-SIM configuration files specifying CF, CR, N_FP, Mu_F, SR, Error_rate, Mu_IS, Std_IS, Hap

## Outputs

- SNV VCFs from FreeBayes and Long Ranger
- SV/deletion call sets from SvABA, Long Ranger, and assembly-based calling
- Supernova2 de novo diploid assemblies (~80% of genome assembled in diploid state under optimal parameters)
- vcfeval, truvari, and VaPoR concordance metrics (precision, recall, F1) per sample and caller
- Simulated Linked-Read FASTQ libraries from LRTK-SIM

## How to apply

Align the Illumina short reads to the hg38 reference with BWA-MEM, call SNVs with FreeBayes and SVs with SvABA, and process the 10x Linked-Read libraries with the Long Ranger pipeline to obtain SNV and deletion calls. For the assembly-based route, build de novo diploid assemblies with Supernova2 and call variants directly from the assembly. Benchmark SNV call sets against GIAB with RTG vcfeval using genotype-aware matching, and SV call sets against GIAB Tier 1 SV regions with truvari; validate assembly-derived SVs by aligning them to PacBio CCS long reads with VaPoR. To simulate Linked-Read libraries, tune LRTK-SIM config parameters: fragment coverage (CF, optimal 332x–823x, no larger than 1,000x at C = 56x), per-fragment read coverage (CR), mean fragment length (Mu_F, optimal WμFL ~50–150 kb), read length (SR), and Hap=2 for diploid simulation. Compile per-sample precision, recall, and F1 by callset and caller into concordance tables for each sample.

## Related tools

- **Long Ranger** (Reference-aligned 10x Linked-Read pipeline producing SNV and deletion calls from Chromium libraries) — https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger
- **Supernova2** (De novo diploid whole-genome assembler for the 10x data; the assembly is also the substrate for assembly-based variant calling) — https://support.10xgenomics.com/de-novo-assembly/software/overview/welcome
- **LRTK-SIM** (Simulator that follows the 10x data-generation workflow to produce realistic Linked-Read FASTQs for parameter-space exploration) — https://github.com/zhanglu295/LRTK-SIM
- **gen_fasta.py** (Auxiliary LRTK-SIM script that builds two haploid FASTA inputs by inserting SNVs from a VCF into the reference genome) — https://github.com/zhanglu295/LRTK-SIM
- **BWA-MEM** (Aligns NA12878/NA24385 Illumina short reads to the hg38 reference prior to FreeBayes/SvABA calling)
- **FreeBayes** (Calls SNVs from the BWA-MEM Illumina alignments for the reference-based comparison arm)
- **SvABA** (Calls SVs from the same BWA-MEM Illumina alignments)
- **RTG vcfeval** (Genotype-aware benchmarking of SNV call sets against GIAB truth sets)
- **truvari** (Benchmarks SV call sets against GIAB Tier 1 SV regions)
- **VaPoR** (Validates assembly-derived SV calls by aligning them to PacBio CCS long reads)
- **Evaluate_diploid_assembly** (Notebook-based code for comparing diploid assemblies of different parameters) — https://github.com/zhanglu295/Evaluate_diploid_assembly

## Examples

```
python LRTK-SIM.py ./diploid_config
```

## Evaluation signals

- LRTK-SIM output FASTQs must be accepted by the official 10x Long Ranger and Supernova pipelines
- vcfeval, truvari, and VaPoR concordance statistics should reproduce the GIAB-comparison values in Supplementary Tables S7–S14 for both NA12878 and NA24385
- Reference-based Illumina and 10x SNV call sets should show comparable accuracy, and both higher accuracy than assembly-derived SNV calls
- Simulated FASTQs follow 10x naming: <lib>_S1_L001_R1_001.fastq.gz and <lib>_S1_L001_R2_001.fastq.gz
- Physical fragment coverage CF should fall within 332x–823x (never >1,000x at C = 56x) for assemblies of acceptable quality

## Limitations

- gen_fasta.py supports SNVs only for constructing diploid reference sequences in this version
- No alternative assembler comparable to Supernova2 was available, so the fragment-length effect cannot be reasoned about independently
- Harm from extremely long fragments (loss of barcode specificity over repeats) is a proposed, unverified explanation
- The 10x Chromium system captures only ~20–40% of the original long-fragment sequence, and the ~2x cost increase over standard Illumina is an estimate, not a measured value

## Evidence

- [abstract] We prepared and sequenced eight 10x libraries with a diverse set of parameters from standard cell lines NA12878 and NA24385: "We prepared and sequenced eight 10x libraries with a diverse set of parameters from standard cell lines NA12878 and NA24385"
- [intro] Supernova2 de novo assembly emphasis: "These simulated and real data sets were then used to produce de novo assemblies, with an emphasis on the performance of 10x's Supernova2"
- [discussion] GIAB comparison of all calls: "All calls were compared to the GIAB benchmark."
- [readme] LRTK-SIM output accepted by official 10x pipelines: "The simulated data have been tested and accepted by 10X official pipelines [Long Ranger]"
- [methods] gen_fasta.py builds haploid FASTA inputs from VCF and reference: "`gen_fasta.py` can generate two haploid fasta files with variants from vcf and reference genome sequence."
