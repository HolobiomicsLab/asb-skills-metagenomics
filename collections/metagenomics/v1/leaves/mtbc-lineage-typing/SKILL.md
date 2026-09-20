---
name: mtbc-lineage-typing
description: Use when raw mycobacterial (MTBC or NTM) sequencing reads need species/lineage
  calls, when reproducing the MTBC_A2/Myco_Retro classification benchmark, or when
  Mykrobe lineage calls must be checked for concordance against KvarQ.
when_to_use_negative:
- The goal is antibiotic-resistance prediction rather than species/lineage identification
  — this skill covers only the speciation output of Mykrobe, not the AMR report
- Input samples are Staphylococcus aureus, Shigella, or Salmonella — Mykrobe's TB
  lineage panel does not apply to these species
- You only have assembled contigs or variant VCFs and no raw reads — Mykrobe predictor
  takes raw sequence data as input
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Mykrobe predictor
  role: Assigns phylo_group, species, and lineage calls from raw reads using the 202010
    probe panel with --min_depth 5 and --min_variant_conf 10
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe parser
  role: Parses Mykrobe JSON output into speciation columns (phylo_group, species,
    lineage with depths and percent coverage); removes NTM-detected samples for QC
  repo: https://github.com/phac-nml/mykrobe-parser
- name: KvarQ
  role: Provides the precomputed reference lineage calls on MTBC_A2 against which
    Mykrobe lineage concordance is computed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:256c13393aa77150210855f9a1adfdbc6aff36e99182380c1423acfebdd48f34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - KvarQ
  - Mykrobe parser
  - Mykrobe predictor
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# mtbc-lineage-typing

## Summary

Species and lineage classification of Mycobacterium tuberculosis complex and non-tuberculous mycobacteria isolates using Mykrobe predictor's de Bruijn-graph probe panel, validated on the MTBC_A2 (1,157 MTBC isolates) and Myco_Retro (147 mycobacterial isolates) datasets with KvarQ lineage calls as reference.

## When to use

Use when raw mycobacterial (MTBC or NTM) sequencing reads need species/lineage calls, when reproducing the MTBC_A2/Myco_Retro classification benchmark, or when Mykrobe lineage calls must be checked for concordance against KvarQ.

## When NOT to use

- The goal is antibiotic-resistance prediction rather than species/lineage identification — this skill covers only the speciation output of Mykrobe, not the AMR report
- Input samples are Staphylococcus aureus, Shigella, or Salmonella — Mykrobe's TB lineage panel does not apply to these species
- You only have assembled contigs or variant VCFs and no raw reads — Mykrobe predictor takes raw sequence data as input

## Inputs

- Raw paired-end Illumina sequencing reads (FASTQ, e.g. gzipped) for MTBC_A2 (1,157 MTBC isolates) and Myco_Retro (147 mycobacterial isolates) from ENA/NCBI SRA
- Precomputed KvarQ lineage calls for the MTBC_A2 isolates
- Mykrobe 202010 species/lineage probe panel

## Outputs

- Per-isolate species/lineage call table with phylo_group, species, and lineage calls plus depths and percent coverage (e.g. Mykrobe parser output-data.csv)
- species_lineage_concordance.csv with MTBC/NTM misclassification counts and Mykrobe-vs-KvarQ lineage concordance rate
- Per-sample Mykrobe JSON result files

## How to apply

Download raw reads for MTBC_A2 and Myco_Retro from ENA/NCBI SRA, then run Mykrobe predictor per sample with --format , the --panel 202010 probe set, and default QC thresholds --min_depth 5 and --min_variant_conf 10. Parse the JSON outputs to extract phylo_group, species, and lineage calls with their depths and percent coverage (Mykrobe parser's output-data.csv stores exactly these fields). Cross-tabulate Mykrobe species calls against expected labels (MTBC for MTBC_A2, non-tuberculous mycobacteria for Myco_Retro) and count MTBC/NTM misclassifications; treat Mykrobe-detected NTM samples in MTBC sets as QC removals. Compare Mykrobe lineage assignments on MTBC_A2 against precomputed KvarQ lineage calls and compute the concordance rate, writing per-isolate calls and the concordance summary to species_lineage_concordance.csv.

## Related tools

- **Mykrobe predictor** (Assigns phylo_group, species, and lineage calls from raw reads using the 202010 probe panel with --min_depth 5 and --min_variant_conf 10) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe parser** (Parses Mykrobe JSON output into speciation columns (phylo_group, species, lineage with depths and percent coverage); removes NTM-detected samples for QC) — https://github.com/phac-nml/mykrobe-parser
- **KvarQ** (Provides the precomputed reference lineage calls on MTBC_A2 against which Mykrobe lineage concordance is computed)

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out. --format  --seq reads.fq.gz
```

## Evaluation signals

- Zero samples misclassified between MTBC and NTM on the union of MTBC_A2 and Myco_Retro (only a handful unresolved to species level within MTBC)
- 100% concordance between Mykrobe lineage calls and KvarQ lineage calls on MTBC_A2
- Every Mykrobe JSON output parsed successfully into output-data.csv with non-empty phylo_group, species, and lineage columns including depths and percent coverage
- Samples flagged by Mykrobe as non-tuberculous mycobacteria are excluded as QC removals rather than scored as MTBC failures
- Lineage calls respect QC thresholds: calls supported by sequencing depth >= 5 and variant confidence >= 10

## Limitations

- Only a handful of MTBC samples remain unresolved to species level within the complex, so within-MTBC species resolution is not perfect even though MTBC/NTM discrimination is error-free in this benchmark
- Samples detected as non-tuberculous mycobacteria are removed for quality control, which shrinks the analyzable set rather than being classified further
- Mykrobe parser is only compatible with Mykrobe v0.7.0 and the 201901/202010 panels; other panel versions are not supported

## Evidence

- [readme] NTM QC removal: "Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes."
- [readme] Parser speciation output columns: "Stores Mykrobe's speciation data such as "phylo_group", "species", and "lineage" and their associated depths and percentages of coverage."
- [readme] JSON format requirement: [evidence span withheld — anchor status: not_found]
- [methods] Panel compatibility: "Mykrobe Parser is compatible with [Mykrobe v0.7.0](https://anaconda.org/bioconda/mykrobe/files?version=0.7.0) and the "201901" / "202010" panels."
- [results] Lineage concordance finding: [evidence span withheld — anchor status: not_found]
