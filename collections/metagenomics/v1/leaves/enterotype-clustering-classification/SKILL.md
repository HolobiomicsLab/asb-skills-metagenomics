---
name: enterotype-clustering-classification
description: Use when you need to cluster genus-level gut microbiome profiles (e.g.
  a twin/mother cohort) into enterotypes and test whether CopyRighter GCN correction
  changes sample-to-enterotype assignment.
when_to_use_negative:
- Your input is raw reads or an unclassified OTU table — enterotype clustering requires
  genus-level assignments on Greengenes taxonomy, not OTU IDs alone
- Samples have fewer than 1,000 reads, below the cohort's depth threshold for reliable
  enterotype assignment
- Your data are shotgun metagenomes or non-gut communities where the enterotype framework
  and dominant-genus clustering do not apply
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CopyRighter
  role: Performs phylogenetic-level GCN correction of the 16S profiles whose enterotype
    classification is being compared before and after correction
  repo: github.com/fangly/AmpliCopyrighter
- name: Bio::Community
  role: Perl dependency used by CopyRighter to read and correct the community input
    files (e.g. bc_use_repr_id for mapping OTU IDs to Greengenes IDs)
  repo: github.com/fangly/AmpliCopyrighter
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:e4ff0c6b2fc31a9851c0787ccdc14ac94f47a03786e328f3eb87ad8858a9bff1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - make
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# enterotype-clustering-classification

## Summary

Assigns 16S rRNA amplicon gut microbiome samples to enterotypes via PAM-style clustering on dominant-genus community profiles, and quantifies how gene copy number (GCN) correction with CopyRighter changes that classification. In the CopyRighter twin cohort, uncorrected profiles split into two enterotypes while GCN-corrected profiles are better partitioned into three, with 23.9% of samples misclassified without correction.

## When to use

Use when you need to cluster genus-level gut microbiome profiles (e.g. a twin/mother cohort) into enterotypes and test whether CopyRighter GCN correction changes sample-to-enterotype assignment.

## When NOT to use

- Your input is raw reads or an unclassified OTU table — enterotype clustering requires genus-level assignments on Greengenes taxonomy, not OTU IDs alone
- Samples have fewer than 1,000 reads, below the cohort's depth threshold for reliable enterotype assignment
- Your data are shotgun metagenomes or non-gut communities where the enterotype framework and dominant-genus clustering do not apply

## Inputs

- Genus-level relative abundance table (e.g. QIIME/biom OTU table from twin cohort V1-V2 16S rRNA amplicon data, 280 samples with >=1,000 reads)
- CopyRighter-corrected genus-level abundance profiles from phylogenetic-level GCN correction
- Greengenes OTU taxonomy assignments with unaltered taxonomic strings

## Outputs

- Per-sample enterotype assignments for uncorrected and corrected profiles
- Count/fraction of samples reassigned between enterotypes after GCN correction (paper value: 23.9%)
- Number of supported enterotypes before and after correction (two vs. three)

## How to apply

Start from a genus-level relative abundance table derived from amplicon profiles — here the twin cohort V1-V2 OTU table (153 individuals, 280 samples with at least 1,000 reads) on Greengenes taxonomy — in both uncorrected form and after CopyRighter phylogenetic-level correction, which weights each taxon's read counts by the inverse of its lineage-specific 16S rRNA gene copy number. Run PAM-style clustering on the dominant-genus community composition to assign each sample to an enterotype (Bacteroides-, Prevotella-, Ruminococcus-like types). Cluster both the uncorrected and corrected profiles separately and compare assignments per sample. Quantify the fraction of samples reassigned after correction and examine whether the corrected data support more enterotypes (here three rather than two). Accept the analysis when a substantial reclassification fraction (paper reports 23.9% of samples misclassified without correction) accompanies a more parsimonious, well-supported partitioning of the corrected profiles.

## Related tools

- **CopyRighter** (Performs phylogenetic-level GCN correction of the 16S profiles whose enterotype classification is being compared before and after correction) — github.com/fangly/AmpliCopyrighter
- **Bio::Community** (Perl dependency used by CopyRighter to read and correct the community input files (e.g. bc_use_repr_id for mapping OTU IDs to Greengenes IDs)) — github.com/fangly/AmpliCopyrighter

## Examples

```
copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic -v
```

## Evaluation signals

- Corrected profiles partition into three enterotypes rather than two, reproducing the paper's finding for the twin cohort
- The fraction of samples reassigned between enterotypes after correction is near the reported 23.9%
- All clustered samples meet the >=1,000-read depth threshold (280 samples from 153 individuals)
- Clustering is run on dominant-genus community types from both uncorrected and CopyRighter-corrected profiles, so the before/after assignment comparison is like-for-like
- CopyRighter verbose output (-v) confirms taxa were assigned GCN values as intended before profiles entered clustering

## Limitations

- The GCN correction effects on community structure are non-trivial to predict for communities with tens to thousands of species
- CopyRighter's pre-computed GCN estimates cover Greengenes; other taxonomies (e.g. Silva) are not yet covered, so the clustering input must be on Greengenes
- Enterotype clustering is applied at the genus level and depends on taxa having UNALTERED taxonomic assignments in the input community file

## Evidence

- [abstract] Twin gut microbiomes reclassified into three rather than two enterotypes: "human gut microbiomes from twins were reclassified into three rather than two enterotypes after GCN correction"
- [supplementary] Enterotype classification of the twin cohort at genus level: "Enterotype classification of human gut microbiomes of a twin cohort at the genus level."
- [readme] CopyRighter correction mechanism underlying the changed classification: "CopyRigher corrects microbial relative abundance by applying a weight proportional to the inverse of the estimated copy number to each species."
- [readme] Input taxa must retain unaltered assignments for the correction feeding into clustering: [evidence span withheld — anchor status: external_doc]
