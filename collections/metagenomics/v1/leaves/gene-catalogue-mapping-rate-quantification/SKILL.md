---
name: gene-catalogue-mapping-rate-quantification
description: Use when benchmarking the read-capture efficiency of a gene catalogue
  (e.g., VMGC-90 vs VIRGO) by aligning clean metagenomic reads and computing per-sample
  mapping rates.
when_to_use_negative:
- Estimating per-gene abundances or functional profiles — this skill only computes
  catalogue-level mapping rates, not gene counts or KEGG/CAZy annotations.
- Reporting whole-genome capture — the VMGC README reports a median mapping rate of
  91.7% against the genome collection, which is a different metric from the 74.5%
  VMGC-90 gene-catalogue average.
- Taxonomic composition profiling — use the customized Kraken2/Bracken VMGC SGB database
  pipeline instead of gene-catalogue read mapping.
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0102
tools:
- name: Bowtie2
  role: Aligns each sample's clean reads end-to-end against the VMGC-90 and VIRGO
    indices (--mm, --fast); its alignment summaries are parsed into per-sample mapping
    rates
- name: GNU parallel 20201122
  role: Parallelizes the per-sample Bowtie2 mapping jobs across the 4,472 vaginal
    metagenomic samples
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:74ae3ffc4be0933f3dadceac0319fd30a02598d13ae2df5e58ee7ce32ba96366
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bowtie2
  - GNU parallel 20201122
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gene-catalogue-mapping-rate-quantification

## Summary

Benchmarks the read-capture efficiency of a vaginal gene catalogue by aligning clean vaginal metagenomic reads against competing catalogues (VMGC-90 vs VIRGO) and computing per-sample mapping rates. In this paper it shows VMGC-90 captures 74.5% of reads on average versus 71.7% for VIRGO.

## When to use

Use when benchmarking the read-capture efficiency of a gene catalogue (e.g., VMGC-90 vs VIRGO) by aligning clean metagenomic reads and computing per-sample mapping rates.

## When NOT to use

- Estimating per-gene abundances or functional profiles — this skill only computes catalogue-level mapping rates, not gene counts or KEGG/CAZy annotations.
- Reporting whole-genome capture — the VMGC README reports a median mapping rate of 91.7% against the genome collection, which is a different metric from the 74.5% VMGC-90 gene-catalogue average.
- Taxonomic composition profiling — use the customized Kraken2/Bracken VMGC SGB database pipeline instead of gene-catalogue read mapping.

## Inputs

- VMGC-90 gene catalogue (Zenodo record 10457006)
- VIRGO non-redundant human vaginal gene catalogue (0.95 million genes)
- Clean paired-end reads from 4,472 publicly available vaginal metagenomic samples (Supplementary Table 2; 32 studies, 14 countries)

## Outputs

- Bowtie2 end-to-end alignment summaries per sample per catalogue
- Per-sample mapping rates (percentage of reads aligned) for VMGC-90 and VIRGO
- Catalogue-level average mapping rates and comparison (VMGC-90 ~74.5% vs VIRGO ~71.7%, as in Fig. 6c)

## How to apply

Download the VMGC-90 gene catalogue from Zenodo record 10457006 and the VIRGO non-redundant human vaginal gene catalogue (0.95 million genes), plus clean reads for the 4,472 publicly available vaginal metagenomic samples listed in Supplementary Table 2. Build a Bowtie2 index once for each catalogue, then align every sample's paired clean reads with Bowtie2 in end-to-end mode using the --mm and --fast flags, parallelizing over samples with GNU parallel. Parse each Bowtie2 alignment summary to compute the per-sample mapping rate (percentage of reads aligned) for both catalogues. Average the rates per catalogue across samples spanning 14 countries and compare them; the validation target is reproducing Fig. 6c, where the VMGC-90 average (~74.5%) exceeds the VIRGO average (~71.7%). Apply the identical read set and alignment mode to both catalogues so the comparison is paired and only the reference differs.

## Related tools

- **Bowtie2** (Aligns each sample's clean reads end-to-end against the VMGC-90 and VIRGO indices (--mm, --fast); its alignment summaries are parsed into per-sample mapping rates)
- **GNU parallel 20201122** (Parallelizes the per-sample Bowtie2 mapping jobs across the 4,472 vaginal metagenomic samples)

## Examples

```
find clean_reads/*.1.fq.gz | sed 's/.1.fq.gz//' | parallel -j 10 bowtie2 --mm --fast -x VMGC-90 -1 {}.1.fq.gz -2 {}.2.fq.gz > vmgc90/{}.sam
```

## Evaluation signals

- Average mapping rate to VMGC-90 reproduces ~74.5% across samples from 14 countries, exceeding the ~71.7% achieved by VIRGO (Fig. 6c)
- Both catalogues are scored on the identical 4,472-sample clean-read set, making the VMGC-90 vs VIRGO difference a paired comparison
- Per-sample rates are parsed from Bowtie2 end-to-end alignment summaries, not recomputed from SAM flags with different criteria
- A Bowtie2 index is built once per catalogue and reused, with --mm and --fast flags present in every alignment command
- Per-sample mapping rates fall in [0, 100] and the number of samples matches the 4,472 listed in Supplementary Table 2

## Limitations

- Read-capture efficiency is a coarse completeness metric; the article notes the microbial sequences captured by the VMGC are not complete, especially for vaginal samples from the non-western population
- Do not conflate the gene-catalogue average (74.5% for VMGC-90) with the genome-collection median mapping rate of 91.7% reported in the README — different references and metrics
- Public vaginal metagenome datasets may carry contamination from experimental operations or kits, as flagged in the article's discussion

## Evidence

- [results] yielded an average mapping rate of 74.5% across samples from 14 different countries, slightly higher than the 71.7% achieved by the VIRGO catalogue.: [evidence span withheld — anchor status: not_found]
- [methods] Align each sample's clean reads against both catalogues with Bowtie2 in end-to-end mode using --mm and --fast, parallelizing over samples with GNU parallel.: [evidence span withheld — anchor status: not_found]
- [abstract] We collected a total of 4,472 publicly available vaginal metagenomic samples sourced from the human vagina, spanning 32 studies: "We collected a total of 4,472 publicly available vaginal metagenomic samples sourced from the human vagina, spanning 32 studies"
- [abstract] the human vaginal non-redundant gene catalogue (VIRGO)10, which includes a total of 0.95 million genes: "the human vaginal non-redundant gene catalogue (VIRGO)10, which includes a total of 0.95 million genes"
- [readme] the VMGC demonstrates high efficiency in capturing microbial sequences, with a median mapping rate of 91.7% across 4,472 vaginal metagenomic samples obtained from 14 countries: "the VMGC demonstrates high efficiency in capturing microbial sequences, with a median mapping rate of 91.7% across 4,472 vaginal metagenomic samples obtained from 14 countries"
- [readme] The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006"
