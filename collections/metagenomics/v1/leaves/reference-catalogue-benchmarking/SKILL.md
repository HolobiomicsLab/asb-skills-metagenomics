---
name: reference-catalogue-benchmarking
description: Use when a new reference gene/genome catalogue (e.g. VMGC-90) must be
  quantitatively compared against a prior catalogue (e.g. VIRGO) using read-mapping
  rates over a shared, publicly available metagenomic sample set.
when_to_use_negative:
- Samples are not vaginal metagenomes or are not present in the paper's Supplementary
  Table 2 sample list — the reported 74.5% vs 71.7% rates apply to that 4,472-sample
  vaginal set
- The goal is taxonomic profiling rather than gene-catalogue read capture — use the
  paper's customized Kraken2/Bracken SGB database workflow instead
- A reference catalogue of genomes (not a non-redundant gene catalogue) is being benchmarked
  — mapping to full genome collections (e.g. VMGC MAGs, median mapping rate 91.7%)
  is a different comparison
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Bowtie2
  role: Aligns each sample's clean reads end-to-end (--mm --fast) against both the
    VMGC-90 and VIRGO gene catalogue indices to obtain per-sample mapping rates
- name: GNU parallel
  role: Parallelizes the Bowtie2 alignment over the 4,472 samples for both catalogues
  repo: https://github.com/RChGO/VMGC
- name: bwa-mem2 2.2.1
  role: Alternative aligner used elsewhere in the VMGC pipeline for read-to-contig
    mapping during binning
  repo: https://github.com/RChGO/VMGC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:be874215aecf34de8b2ff31edba71da2b18de88b2ad53b38bd53ba88c03498b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RChGO/VMGC
  tools:
  - Bowtie2
  - GNU parallel
  - bwa-mem2 2.2.1
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# reference-catalogue-benchmarking

## Summary

Benchmarks a newly built reference gene catalogue against an established one by mapping the same set of metagenomic samples to both catalogues and comparing average per-sample mapping rates. Used here to show VMGC-90 captures vaginal metagenome reads more efficiently (~74.5%) than VIRGO (~71.7%).

## When to use

Use when a new reference gene/genome catalogue (e.g. VMGC-90) must be quantitatively compared against a prior catalogue (e.g. VIRGO) using read-mapping rates over a shared, publicly available metagenomic sample set.

## When NOT to use

- Samples are not vaginal metagenomes or are not present in the paper's Supplementary Table 2 sample list — the reported 74.5% vs 71.7% rates apply to that 4,472-sample vaginal set
- The goal is taxonomic profiling rather than gene-catalogue read capture — use the paper's customized Kraken2/Bracken SGB database workflow instead
- A reference catalogue of genomes (not a non-redundant gene catalogue) is being benchmarked — mapping to full genome collections (e.g. VMGC MAGs, median mapping rate 91.7%) is a different comparison

## Inputs

- VMGC-90 non-redundant gene catalogue (Zenodo record 10457006)
- VIRGO human vaginal non-redundant gene catalogue (0.95 million genes)
- Clean paired-end reads (FASTQ) for 4,472 publicly available vaginal metagenomic samples from 32 studies (Supplementary Table 2)

## Outputs

- Per-sample mapping rates (percentage of reads aligned) to VMGC-90 and to VIRGO
- Average mapping rate per catalogue (VMGC-90 ~74.5%, VIRGO ~71.7%) and comparison corresponding to Fig. 6c

## How to apply

Download both the new catalogue (VMGC-90 from Zenodo record 10457006) and the comparator (VIRGO non-redundant gene catalogue, 0.95 million genes), plus the clean reads for the 4,472 publicly available vaginal metagenomic samples listed in Supplementary Table 2. Build Bowtie2 indices for each catalogue, then align every sample's clean reads against both indices with Bowtie2 in end-to-end mode using --mm and --fast, parallelizing over samples with GNU parallel. Parse each Bowtie2 alignment summary to compute the per-sample mapping rate (percentage of reads aligned) for each catalogue, then average mapping rates per catalogue across samples and compare VMGC-90 to VIRGO. The comparison is validated by reproducing the reported relationship in which the average mapping rate to VMGC-90 genes (~74.5%) exceeds that to VIRGO (~71.7%), as shown in Fig. 6c; samples span 14 countries and 32 studies, so per-country subgroup checks can confirm robustness.

## Related tools

- **Bowtie2** (Aligns each sample's clean reads end-to-end (--mm --fast) against both the VMGC-90 and VIRGO gene catalogue indices to obtain per-sample mapping rates)
- **GNU parallel** (Parallelizes the Bowtie2 alignment over the 4,472 samples for both catalogues) — https://github.com/RChGO/VMGC
- **bwa-mem2 2.2.1** (Alternative aligner used elsewhere in the VMGC pipeline for read-to-contig mapping during binning) — https://github.com/RChGO/VMGC

## Examples

```
bowtie2 --mm --fast -x VMGC-90 -1 $s.1.fq.gz -2 $s.2.fq.gz | tee $s.vmgc90.sam | grep 'overall alignment rate'
```

## Evaluation signals

- Average mapping rate to VMGC-90 should reproduce ~74.5% across the samples from 14 countries
- Average mapping rate to VIRGO should reproduce ~71.7%, with VMGC-90 > VIRGO as in Fig. 6c
- Every one of the 4,472 samples must have a parsed Bowtie2 alignment summary for both catalogues before averaging
- Bowtie2 must be run in end-to-end mode with --mm and --fast; changing mode/tuning would make rates incomparable to the reported values

## Limitations

- VMGC's microbial sequence capture is incomplete, especially for vaginal samples from non-western populations
- VMGC lacks fungal genomes from Cladosporium and Malassezia observed via ITS sequencing, so fungal reads may be systematically unmapped
- Potential contamination from experimental operations or kits in the public vaginal metagenome datasets could affect mapping-based summaries

## Evidence

- [abstract] Mapping of vaginal metagenomes against the VMGC-90 gene catalogue yielded an average mapping rate of 74.5% across samples from 14 different countries, slightly higher than the 71.7% achieved by the VIRGO catalogue.: [evidence span withheld — anchor status: not_found]
- [abstract] VIRGO catalogue size: "the human vaginal non-redundant gene catalogue (VIRGO)10, which includes a total of 0.95 million genes"
- [abstract] Sample set description: "We collected a total of 4,472 publicly available vaginal metagenomic samples sourced from the human vagina, spanning 32 studies"
- [intro] VMGC data location: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006"
- [readme] Repository claim on genome mapping: "with a median mapping rate of 91.7% across 4,472 vaginal metagenomic samples obtained from 14 countries"
