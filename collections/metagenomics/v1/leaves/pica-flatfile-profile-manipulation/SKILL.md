---
name: pica-flatfile-profile-manipulation
description: Use when you need to benchmark phenotypic trait prediction on partially
  complete genomes (e.g. metagenomic bins) by generating thinned COG presence/absence
  profiles at controlled completeness levels of 10-90%.
when_to_use_negative:
- The input is already a complete, fully annotated genome — thinning would only measure
  robustness, not real-world bin incompleteness
- Your features are not COG presence/absence profiles compatible with the PICA flat-file
  format (e.g. raw FASTQ reads or gene sequences)
- You need predictions from models outside the PICA SVM plug-in workflow, e.g. the
  newer phenotrex package, whose training/evaluation pipeline is separate
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: PICA
  role: Framework whose flat-file profile format stores the COG presence/absence profiles
    being thinned, and whose phenotypic trait models are evaluated on the incomplete
    profiles
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Consumes the incomplete-profile files as test sets for SVM-based phenotypic
    trait prediction at each completeness level
  repo: github.com/univieCUBE/PICA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:aca76154585940356eff39553e074744605fec7b7a2eff08ef03af4d642a1788
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICA
  - PICA support vector machine plug-in
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# pica-flatfile-profile-manipulation

## Summary

Simulates incomplete genome sequences by randomly removing x% (10-90) of COG entries from eggNOG 4.0-based PICA flat-file presence/absence profiles, with 3 replicates per completeness level, to assess how genome incompleteness affects SVM-based phenotypic trait prediction for metagenomic bins.

## When to use

Use when you need to benchmark phenotypic trait prediction on partially complete genomes (e.g. metagenomic bins) by generating thinned COG presence/absence profiles at controlled completeness levels of 10-90%.

## When NOT to use

- The input is already a complete, fully annotated genome — thinning would only measure robustness, not real-world bin incompleteness
- Your features are not COG presence/absence profiles compatible with the PICA flat-file format (e.g. raw FASTQ reads or gene sequences)
- You need predictions from models outside the PICA SVM plug-in workflow, e.g. the newer phenotrex package, whose training/evaluation pipeline is separate

## Inputs

- eggNOG 4.0 COG presence/absence profiles in PICA flat-file format for genomes with known example traits
- Fixed random seed for reproducible sampling

## Outputs

- 27 incomplete-profile files (9 completeness levels x 3 replicates) named e.g. incomplete_10_1 ... incomplete_90_3 in PICA-compatible flat-file format
- Each file containing exactly (100 - x) percent of the original per-genome COG count, with no COGs outside the original profile

## How to apply

Load the eggNOG 4.0 COG presence/absence profiles for the genomes associated with the example traits in PICA's flat-file profile format. For each removal level x in {10, 20, ..., 90}, randomly sample and delete x percent of the total COG entries in each genome's profile using a fixed random seed for reproducibility. Repeat the random-removal procedure 3 times per completeness level to yield 3 independent replicates, writing each incomplete profile to a file named by completeness level and replicate (e.g. incomplete_90_1) while keeping the PICA-compatible format so the files can be consumed directly by the PICA SVM plug-in as test sets. The rationale is to emulate metagenomic bins, which are often incomplete, and to determine the completeness threshold below which SVM-based trait prediction degrades; the underlying study found most traits can be reliably predicted in only 60-70% complete genomes.

## Related tools

- **PICA** (Framework whose flat-file profile format stores the COG presence/absence profiles being thinned, and whose phenotypic trait models are evaluated on the incomplete profiles) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Consumes the incomplete-profile files as test sets for SVM-based phenotypic trait prediction at each completeness level) — github.com/univieCUBE/PICA

## Examples

```
python simulate_incompleteness.py --profile profiles.flat --levels 10,20,30,40,50,60,70,80,90 --replicates 3 --seed 42 --outdir incomplete_profiles
```

## Evaluation signals

- Exactly 27 incomplete-profile files exist (9 completeness levels x 3 replicates)
- Each genome's incomplete profile contains exactly (100 - x) percent of its original COG count for the corresponding level x
- No COGs appear in the incomplete profiles that were not present in the original profile
- File naming follows the completeness-level/replicate convention (e.g. incomplete_90_1, incomplete_90_2, incomplete_90_3)
- Re-running with the fixed random seed reproduces identical thinned profiles

## Limitations

- Random COG removal assumes features are lost independently and uniformly; real metagenomic bins may lose genes non-randomly (e.g. linked genomic regions)
- Most traits are only reliably predicted in 60-70% complete genomes, so predictions below that threshold may be unreliable
- Performance for further ecologically important traits is pending availability of sufficient training data (ongoing work per the article's discussion)

## Evidence

- [results] Incomplete test genomes were simulated by random removal of x percent of the total number of COGs per genome: [evidence span withheld — anchor status: not_found]
- [abstract] Most traits reliably predicted at 60-70% completeness: "Most of the traits can be reliably predicted in only 60-70% complete genomes."
- [abstract] SVM plug-in applicability to incomplete genome sequences: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [readme] phenotrex succeeds PICA: "Please checkout the new `phenotrex` package, which succeeds PICA:
https://github.com/univieCUBE/phenotrex"
