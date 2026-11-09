---
name: de-bruijn-graph-baseline-variant-calling
description: Use when benchmarking a reference-free variant caller against the de
  Bruijn graph on simulated or real WGS reads, needing an independent baseline call
  set for hap.py validation.
when_to_use_negative:
- You need to call variants from a BWT/eBWT of the read collection alone without a
  de Bruijn graph — that is ebwt2InDel's role, not the baseline's
- Input is not raw reads (e.g. already an aligned BAM or an existing VCF call set)
  — DISCOSNP++ in this workflow operates on raw read sets
- The comparison targets INDEL detection with the parallel pipeline's external pre-processing
  tool, which does not take into account the presence of INDELs in the read set
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: DISCOSNP++
  role: De Bruijn graph-based variant caller run on the same read set as the baseline
    comparison
- name: snp2vcf
  role: Pipeline (seqtk -> bwa mem -> sam2vcf) converting DISCOSNP++ variant-fragment
    fasta output to VCF for validation
- name: hap.py
  role: Validates the baseline VCF against the ground-truth variant VCF to compute
    SNP/INDEL sensitivity and precision
  repo: Illumina/hap.py
- name: SimSeq
  role: Simulates the synthetic 30x Human chromosome 1 read set on which the baseline
    is evaluated
- name: filter_snp
  role: Post-processing filter applied to keep only variants supported by at least
    m reads (e.g. 5) in the comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03586-3
    title: Variable-order reference-free variant discovery with the Burrows-Wheeler
      Transform
  source_content_hash: sha256:f887cae4dcd64c27114781d81cf6a599fe99fc5e5f87f10cfe2cf878a6b1c61c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Illumina/hap.py
  tools:
  - BCR
  - DISCOSNP++
  - SimSeq
  - bwa mem
  - ebwt2InDel
  - filter_snp
  - hap.py
  - sam2vcf
  - seqtk
  - snp2vcf
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# de-bruijn-graph-baseline-variant-calling

## Summary

Run DISCOSNP++, the de Bruijn graph-based variant caller, on the same read set as the eBWT-based tool to provide the state-of-the-art baseline for SNP/INDEL sensitivity and precision comparison.

## When to use

Use when benchmarking a reference-free variant caller against the de Bruijn graph state-of-the-art on simulated or real WGS reads, needing an independent baseline call set for hap.py validation.

## When NOT to use

- You need to call variants from a BWT/eBWT of the read collection alone without a de Bruijn graph — that is ebwt2InDel's role, not the baseline's
- Input is not raw reads (e.g. already an aligned BAM or an existing VCF call set) — DISCOSNP++ in this workflow operates on raw read sets
- The comparison targets INDEL detection with the parallel pipeline's external pre-processing tool, which does not take into account the presence of INDELs in the read set

## Inputs

- FASTA/FASTQ read collection (e.g. 100 bp SimSeq-simulated reads from bi-allelic Human chromosome 1 at 30x, HiSeq error profile)
- Reference genome assembly (hg38) for VCF conversion of variant fragments
- Ground-truth variant VCF (heterozygous variations from the 1000 Genomes Project phase 3 release 20130502)

## Outputs

- Variant fragment .fasta output in KisSNP2 format (contexts surrounding identified SNPs/indels)
- VCF call set after snp2vcf conversion (seqtk -> bwa mem -> sam2vcf) and duplicate removal
- hap.py-derived SNP and INDEL sensitivity and precision metrics (sensitivity_comparison.csv)

## How to apply

Run DISCOSNP++ directly on the same preprocessed read set (e.g. SimSeq-simulated 100 bp reads at 30x coverage from bi-allelic Human chromosome 1, filtered for reads containing 'N' and augmented with reverse-complements) that was given to the tool under test. Like the eBWT tool, DISCOSNP++ outputs variant fragments in KisSNP2-style fasta format; convert these to VCF with the snp2vcf pipeline (seqtk with fake base qualities -> bwa mem against hg38 -> sam2vcf), since both tools detect variants in the dataset and its reverse-complement, then remove duplicate VCF entries from variants found on both strands. Validate the resulting VCF with hap.py against the ground-truth variant VCF (heterozygous variants from the 1000 Genomes phase 3 release) exactly as done for the tool under test, so sensitivity/precision are computed under identical preprocessing, VCF conversion, and validation conditions. On the 30x synthetic chromosome 1 dataset, the baseline should yield ~71% SNP and ~51% INDEL sensitivity for comparison against ebwt2InDel's ~83% / ~72%.

## Related tools

- **DISCOSNP++** (De Bruijn graph-based variant caller run on the same read set as the baseline comparison)
- **snp2vcf** (Pipeline (seqtk -> bwa mem -> sam2vcf) converting DISCOSNP++ variant-fragment fasta output to VCF for validation)
- **hap.py** (Validates the baseline VCF against the ground-truth variant VCF to compute SNP/INDEL sensitivity and precision) — Illumina/hap.py
- **SimSeq** (Simulates the synthetic 30x Human chromosome 1 read set on which the baseline is evaluated)
- **filter_snp** (Post-processing filter applied to keep only variants supported by at least m reads (e.g. 5) in the comparison)

## Examples

```
hap.py truth.vcf discosnp_query.vcf -f confident.bed -o baseline_prefix -r hg38.fa
```

## Evaluation signals

- On 30x synthetic Human chromosome 1, hap.py-derived baseline SNP sensitivity should approximate ~71% and INDEL sensitivity ~51%, matching the paper's reported DISCOSNP++ results
- The baseline VCF contains no duplicate entries after removing variants found twice (forward and reverse strands)
- Both tools (baseline and ebwt2InDel) are validated with hap.py against the same ground-truth VCF, so metrics are directly comparable in sensitivity_comparison.csv
- The baseline read set is identical to the tool-under-test's input: reads containing 'N' removed and reverse-complements added

## Limitations

- The state-of-the-art competitor tool is referenced only descriptively (de Bruijn graph-based) in parts of the text — the reproducibility gap must be closed by explicitly using DISCOSNP++
- Real-data evaluation relies on an approximated ground truth, lowering precision estimates for both tools including the baseline
- No software version numbers are stated for DISCOSNP++ in the paper, so exact version choice is not constrained by the article

## Evidence

- [results] DISCOSNP++ baseline: "We compared our EBWT2INDEL with DISCOSNP++ [8], that is an improvement of the DISCOSNP [1, 5] algorithm"
- [methods] snp2vcf conversion: "We call **snp2vcf** the pipeline **seqtk -> bwa-mem -> sam2vcf**."
- [results] duplicate removal: [evidence span withheld — anchor status: not_found]
- [results] hap.py validation: "and validated the calls using hap.py"
- [abstract] baseline sensitivity numbers: "These percentages considerably improve the 71% of SNPs and 51% of INDELs found by the state-of-the art tool based on de Bruijn graphs"
- [readme] hap.py usage: [evidence span withheld — anchor status: external_doc]
- [results] VCF conversion via hg38: [evidence span withheld — anchor status: not_found]
