---
name: differential-abundance-benchmarking
description: Use when comparing phenotype-enrichment methods for species/pathway abundance
  tables (16S or metagenomic), especially with limited cohort sizes, or when reproducing
  the VTwins-vs-conventional sensitivity/10-fold-sample-size benchmark.
when_to_use_negative:
- Input is raw counts or already-normalized compositions not suited to decostand 'total'
  relative-abundance normalization, or the table is not a sample-by-feature data frame
  as required by pair_find
- Fewer than 10 eligible sample pairs exist across the two phenotype groups — pair_find
  returns nothing in that case
- The goal is generic multi-software benchmarking with tools other than VTwins beyond
  the article's 16-software benchmark scope, or samples are not clustered and ordered
  by grp1/grp2 as the method requires
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: VTwins
  role: 'Core method: transforms the original cohort into a paired cohort of ''Twin''
    samples and performs phenotype-enriched feature detection via pair_find'
  repo: https://github.com/mengqingren/VTwins
- name: vegan
  role: Normalizes the abundance matrix to relative abundances with decostand(method='total',
    margin=1)
- name: tidyverse
  role: Supporting R data-wrangling dependency (tidyverse_1.3.1) used in the analysis
    script
- name: curatedMetagenomicData
  role: Supplies the empirical benchmark dataset (ZellerG_2014.metaphlan_bugs_list.stool)
    as a phyloseq object
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.scib.2023.10.024
    title: 'VTwins: inferring causative microbial features from metagenomic data of
      limited samples'
  source_content_hash: sha256:f30e7682d2cb669d866d4e1d350cecfce6c68a4afade1759105d50f1a4410baa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mengqingren/VTwins
  tools:
  - R
  - VTwins
  - curatedMetagenomicData
  - tidyverse
  - vegan
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# differential-abundance-benchmarking

## Summary

Benchmark causal-feature detection in metagenomic data by running VTwins, which pairs samples across phenotypes to eliminate confounder effects, against a conventional unpaired differential-abundance approach on simulated and empirical datasets. VTwins reportedly matches conventional recall with a 10-fold smaller sample size.

## When to use

Use when comparing phenotype-enrichment methods for species/pathway abundance tables (16S or metagenomic), especially with limited cohort sizes, or when reproducing the VTwins-vs-conventional sensitivity/10-fold-sample-size benchmark.

## When NOT to use

- Input is raw counts or already-normalized compositions not suited to decostand 'total' relative-abundance normalization, or the table is not a sample-by-feature data frame as required by pair_find
- Fewer than 10 eligible sample pairs exist across the two phenotype groups — pair_find returns nothing in that case
- The goal is generic multi-software benchmarking with tools other than VTwins beyond the article's 16-software benchmark scope, or samples are not clustered and ordered by grp1/grp2 as the method requires

## Inputs

- Relative-abundance data frame with features as columns and samples as rows (metagenomic or 16S; e.g. mock runif matrix or curatedMetagenomicData phyloseq object like ZellerG_2014.metaphlan_bugs_list.stool)
- Phenotype data frame with columns `id` (sample id) and `grp` (grp1/grp2)

## Outputs

- Results.csv containing Species, Enriched, Incre./Decre.aveRank.P with FDR columns, rank metrics, Ctlmean, Dismean, and log2FC per feature
- ShuffleWstat, BoundarySample, and BoundaryPair output files in the working directory

## How to apply

Generate a mock relative-abundance data frame (e.g. runif(min=1e-5, max=1), 120 samples x 10 features) and normalize with vegan::decostand(method='total', margin=1). Build a phenodata frame with columns `id` and `grp` (grp1=control, grp2=disease), keeping sample order identical to the abundance matrix. Run VTwins::pair_find with k='euclidean', Cut_pair=25, method_choose='Permutation', ShuffleTime=10000, DownPercent=0.2, Uppercent=0.8, and PvalueCutoff=0.01, saving ShuffleWstat, BoundarySample, and BoundaryPair. Compute log2FC as log2(Dismean/Ctlmean) on the result and export the full table (Incre./Decre.aveRank.P with their FDR columns) via write.csv as Results.csv. Judge success by confirming Results.csv exists and that significant features at PvalueCutoff=0.01 reproduce the reported VTwins sensitivity advantage over the conventional approach.

## Related tools

- **VTwins** (Core method: transforms the original cohort into a paired cohort of 'Twin' samples and performs phenotype-enriched feature detection via pair_find) — https://github.com/mengqingren/VTwins
- **vegan** (Normalizes the abundance matrix to relative abundances with decostand(method='total', margin=1))
- **tidyverse** (Supporting R data-wrangling dependency (tidyverse_1.3.1) used in the analysis script)
- **curatedMetagenomicData** (Supplies the empirical benchmark dataset (ZellerG_2014.metaphlan_bugs_list.stool) as a phyloseq object)

## Examples

```
res <- pair_find(data=dataset.normalized, phenodata=phe_data, k="euclidean", Cut_pair=25, method_choose="Permutation", ShuffleTime=10000, DownPercent=0.2, Uppercent=0.8, PvalueCutoff=0.01)
```

## Evaluation signals

- Results.csv exists in the working directory after the run and contains Species, Enriched, Decre.aveRank.P, Decre.aveRank.P.FDR, Incre.aveRank.P, Incre.aveRank.P.FDR, Ctlmean, Dismean, and a log2FC column computed as log2(Dismean/Ctlmean)
- Significant features are called at PvalueCutoff=0.01 using Decre.aveRank.P for disease-enriched and Incre.aveRank.P for control-enriched features
- ShuffleWstat, BoundarySample, and BoundaryPair files are produced in SavePath
- At the same PvalueCutoff, VTwins calls on the simulated data match or exceed conventional-approach recall of causative features, consistent with the reported 10-fold sample-size reduction
- Sample order is identical between the abundance data frame and phenodata, with samples clustered by grp1 then grp2, and at least 10 pairs are formed

## Limitations

- If sample pairs are fewer than 10, pair_find returns nothing
- Sample order must be identical in the abundance data frame and phenodata, and samples must be clustered and placed according to grp1/grp2 in the phenotype data, or results are invalid
- Cut_pair=25 governs the redundant-pair cutoff: below 25 pairs the method switches test strategy; method_choose must be one of 'Wilcox' or 'Permutation' for more than 25 pairs
- k must match a method accepted by the R dist function (e.g. 'euclidean')

## Evidence

- [abstract] VTwins outperforms the conventional approach in the sensitivity of identifying causative features and only requires a 10-fold reduced sample size: "The results show that VTwins outperforms the conventional approach in the sensitivity of identifying causative features and only requires a 10-fold reduced sample size"
- [methods] decostand normalization step: "dataset.normalized <- decostand(dataset,method = "total",1) #normalization for fetures like species's relative abundance"
- [readme] README basic-usage parameters and sample-order requirement: "Must keep the `Same Order` of samples in  relative abundance dataframe and phenotype dataframe. And we also keep the samples `clustered and placed` according the `grp1 and grp2`"
- [readme] README note on minimum pair count: "**Note:** if the sample pairs are less than 10, it will return nothing."
