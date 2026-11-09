---
name: spearman-correlation-agreement-analysis
description: Use when two per-pair strain-relatedness scores (e.g., SynTracker APSS
  and inStrain popANI) have been computed on the same sample pairs and you must test
  whether the two methods agree, or calibrate a joint same-strain cutoff.
when_to_use_negative:
- Only one of the two scores exists (e.g., APSS alone with no matched inStrain popANI
  values) — correlation cannot be computed on unmatched vectors
- You need per-region raw synteny scores rather than per-pair agreement — use 'synteny_scores_per_region.csv'
  directly instead
- The comparison is not paired (different genome sets or sample pairs per tool), making
  rank correlation between scores meaningless
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SynTracker
  role: Produces per-pair APSS (Average Pairwise Synteny Score) values from genome-synteny
    analysis, exported as 'avg_synteny_scores_all_regions' tables that serve as one
    vector of the correlation
  repo: https://github.com/leylabmpi/SynTracker
- name: inStrain
  role: Produces the SNP-based popANI values for the same genome pairs, serving as
    the second vector of the correlation and the joint same-strain cutoff (>0.99999)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:7c9006f449207d52507e30550e1c1630b3ac97e7ef5901eb54e95cdebc3109a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - SynTracker
  - inStrain
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spearman-correlation-agreement-analysis

## Summary

Cross-method agreement analysis that quantifies concordance between the synteny-based APSS score from SynTracker and the SNP-based popANI score from inStrain using Spearman rank correlation (e.g., rho = 0.985 over 66 N. gonorrhoeae comparisons), validating that two orthogonal strain-comparison metrics rank strain pairs consistently.

## When to use

Use when two per-pair strain-relatedness scores (e.g., SynTracker APSS and inStrain popANI) have been computed on the same sample pairs and you must test whether the two methods agree, or calibrate a joint same-strain cutoff.

## When NOT to use

- Only one of the two scores exists (e.g., APSS alone with no matched inStrain popANI values) — correlation cannot be computed on unmatched vectors
- You need per-region raw synteny scores rather than per-pair agreement — use 'synteny_scores_per_region.csv' directly instead
- The comparison is not paired (different genome sets or sample pairs per tool), making rank correlation between scores meaningless

## Inputs

- SynTracker APSS table 'avg_synteny_scores_all_regions.csv' (one APSS value per pair of genomes/assemblies)
- SynTracker APSS table 'avg_synteny_scores_200_regions.csv' (200-region subsampling per pair)
- inStrain popANI values for every isolate pair (from the four SRA collections, Supplementary Tables 2-5)

## Outputs

- Spearman rho (and p-value) quantifying APSS-popANI agreement over matched pairwise comparisons
- Per-species same-strain classification tables combining APSS > 0.955 and popANI > 0.99999 cutoffs
- Validation summary listing expected counts per species (66 N. gonorrhoeae rows, 185 hypermutator E. coli rows)

## How to apply

Extract per-pair APSS values from the SynTracker 'avg_synteny_scores_all_regions' tables (or up to 200-region subsampled tables, which yield maximal average sensitivity of 98.6% and specificity of 97%) and match them to popANI values from inStrain run on every isolate pair. Ensure both score vectors cover exactly the same pairwise comparisons (e.g., 66 N. gonorrhoeae pairs, 185 hypermutator E. coli pairs). Compute the Spearman rank correlation between APSS and popANI as a rank-based agreement statistic, then apply joint same-strain cutoffs (APSS > 0.955 and popANI > 0.99999) to produce per-species classification tables. Concordance is judged by both a high Spearman rho (expected 0.985 for the N. gonorrhoeae benchmark) and agreement of the binary same-strain calls between the two tools.

## Related tools

- **SynTracker** (Produces per-pair APSS (Average Pairwise Synteny Score) values from genome-synteny analysis, exported as 'avg_synteny_scores_all_regions' tables that serve as one vector of the correlation) — https://github.com/leylabmpi/SynTracker
- **inStrain** (Produces the SNP-based popANI values for the same genome pairs, serving as the second vector of the correlation and the joint same-strain cutoff (>0.99999))

## Examples

```
from scipy.stats import spearmanr; rho, p = spearmanr(df['APSS'], df['popANI'])  # expect rho=0.985 over 66 N. gonorrhoeae pairs
```

## Evaluation signals

- Spearman rho between APSS and popANI over the 66 N. gonorrhoeae comparisons must equal 0.985
- Each per-species classification table contains all pairwise comparisons for its species (e.g., 66 rows for N. gonorrhoeae, 185 rows for hypermutator E. coli)
- All 185 hypermutator E. coli pairs are called same strain by both tools (APSS > 0.955 and popANI > 0.99999)
- S. rimosus library pairs show a wide APSS range despite clonal (near-identical) popANI values, confirming the two metrics rank differently there
- APSS values fall within the valid score range and popANI within its own valid range before correlation

## Limitations

- Strain delineation thresholds (inStrain popANI 99.999%, APSS cutoffs) are subjective and species-dependent; the APSS cutoff was based on averaging different per-species cutoffs and may not be optimal for all settings
- Classification of genomes into strains is inherently subjective; the authors recommend preferring continuous genomic difference measures over binary same/different calls
- Small numbers of regions per pairwise comparison mostly result in lower accuracy; a trade-off exists between accuracy and sample size

## Evidence

- [full_text] Spearman rho = 0.985 between APSS and popANI over the 66 N. gonorrhoeae comparisons: [evidence span withheld — anchor status: not_found]
- [readme] The table containing the APSS using all regions (without subsampling) is called avg_synteny_scores_all_regions.csv.: "The table containing the APSS using all regions (without subsampling) 
is called `avg_synteny_scores_all_regions.csv`."
- [discussion] Maximal average sensitivity and specificity with 200 regions per pairwise comparison.: "maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison"
- [discussion] SynTracker outperformed MIDAS and StrainPhlAn and achieved a slightly better performance score than InStrain in terms of accuracy.: "SynTracker outperformed MIDAS35 and StrainPhlAn36 and achieved a slightly better performance score than InStrain28 in terms of accuracy"
