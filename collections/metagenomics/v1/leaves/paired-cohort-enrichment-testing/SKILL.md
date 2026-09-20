---
name: paired-cohort-enrichment-testing
description: Use when you have a relative abundance feature table (species or pathways
  from metagenomic/16S sequencing) with a two-group phenotype (ctrl vs disease) and
  limited samples, and need sensitive phenotype-enriched feature identification.
when_to_use_negative:
- Sample pairs formed are fewer than 10 — the README notes 'if the sample pairs are
  less than 10, it will return nothing'
- Input is raw sequencing counts or unnormalized abundance — the method expects a
  normalized relative abundance data frame (decostand 'total')
- Samples in the abundance data frame and phenodata are not in the same order — the
  method requires 'Same Order' and clustering by grp1/grp2
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3053
tools:
- name: VTwins
  role: Core R package performing the twin-pairing and phenotype-enrichment testing
    via pair_find
  repo: https://github.com/mengqingren/VTwins
- name: vegan
  role: decostand(method='total', MARGIN=1) normalizes the abundance data frame before
    pairing
- name: tidyverse
  role: R dependency (version 1.3.1) loaded for data handling in the pipeline
- name: curatedMetagenomicData
  role: Bioconductor source of empirical test datasets (e.g. ZellerG_2014.metaphlan_bugs_list.stool)
- name: VTwins.Linux
  role: Standalone Linux R-script version of the same pairing/enrichment workflow
    (PairFind.Linux.R)
  repo: https://github.com/mengqingren/VTwins.Linux
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.scib.2023.10.024
    title: 'VTwins: inferring causative microbial features from metagenomic data of
      limited samples'
  source_content_hash: sha256:df192bfab04377174f776963cfc6df113800594daa87b51943d2b025357872fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mengqingren/VTwins
  tools:
  - R
  - VTwins
  - VTwins.Linux
  - curatedMetagenomicData
  - tidyverse
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# paired-cohort-enrichment-testing

## Summary

VTwins-based paired-cohort enrichment testing transforms an unpaired metagenomic cohort (species or pathway relative abundances) into matched 'twin' sample pairs to identify phenotype-enriched causative features with higher sensitivity than conventional approaches, achieving comparable recall with a 10-fold smaller sample size.

## When to use

Use when you have a relative abundance feature table (species or pathways from metagenomic/16S sequencing) with a two-group phenotype (ctrl vs disease) and limited samples, and need sensitive phenotype-enriched feature identification.

## When NOT to use

- Sample pairs formed are fewer than 10 — the README notes 'if the sample pairs are less than 10, it will return nothing'
- Input is raw sequencing counts or unnormalized abundance — the method expects a normalized relative abundance data frame (decostand 'total')
- Samples in the abundance data frame and phenodata are not in the same order — the method requires 'Same Order' and clustering by grp1/grp2

## Inputs

- Relative abundance data frame (rows = samples, columns = features, e.g. species or pathways from metaphlan bugs_list or 16S data)
- Phenotype data frame with two columns named 'id' (sample ID) and 'grp' (grp1 = control, grp2 = disease)

## Outputs

- Results.csv containing Species, Enriched group, Incre./Decre.aveRank.P and their FDR columns, Decreasing/Increasing.Rank.Average, Ctlmean, Dismean, and computed log2FC
- ShuffleWstat, BoundarySample, and BoundaryPair output files from pair_find

## How to apply

Normalize the relative abundance data frame with vegan::decostand(method='total', margin=1), then build a phenodata frame with columns 'id' and 'grp' (grp1=control, grp2=disease), keeping sample order identical and samples clustered by grp1/grp2. Call VTwins::pair_find with k='euclidean', Cut_pair=25, method_choose='Permutation', ShuffleTime=10000, DownPercent=0.2, Uppercent=0.8, and PvalueCutoff (e.g. 0.01); pairs fewer than 10 return nothing, and pairs exceeding Cut_pair are tested by permutation. Compute log2FC as log2(Dismean/Ctlmean), interpret Decre.aveRank.P for disease-enriched and Incre.aveRank.P for control-enriched features (with their FDR columns), and export the result table via write.csv(res, 'Results.csv', row.names=F).

## Related tools

- **VTwins** (Core R package performing the twin-pairing and phenotype-enrichment testing via pair_find) — https://github.com/mengqingren/VTwins
- **vegan** (decostand(method='total', MARGIN=1) normalizes the abundance data frame before pairing)
- **tidyverse** (R dependency (version 1.3.1) loaded for data handling in the pipeline)
- **curatedMetagenomicData** (Bioconductor source of empirical test datasets (e.g. ZellerG_2014.metaphlan_bugs_list.stool))
- **VTwins.Linux** (Standalone Linux R-script version of the same pairing/enrichment workflow (PairFind.Linux.R)) — https://github.com/mengqingren/VTwins.Linux

## Examples

```
res <- pair_find(data=dataset.normalized, phenodata=phe_data, k="euclidean", Cut_pair=25, method_choose="Permutation", ShuffleTime=10000, DownPercent=0.2, Uppercent=0.8, PvalueCutoff=0.01); res$log2FC = log2(as.numeric(res$Dismean)/as.numeric(res$Ctlmean)); write.csv(res,"Results.csv",row.names=F)
```

## Evaluation signals

- Results.csv is written to the working directory and contains the Incre./Decre.aveRank.P columns plus their FDR columns
- ShuffleWstat, BoundarySample, and BoundaryPair output files exist after pair_find runs
- log2FC column equals log2(Dismean/Ctlmean) for every feature row
- Enriched-feature calls at PvalueCutoff=0.01 reproduce the reported VTwins sensitivity advantage (10-fold reduced sample size) over the conventional approach on the mock 120-sample × 10-feature runif dataset with set.seed(12345)
- Feature table was normalized with decostand(method='total') and sample order matches phenodata exactly

## Limitations

- If the sample pairs are less than 10, pair_find returns nothing
- Sample order in the relative abundance data frame and phenodata must be identical, with samples clustered and placed according to grp1/grp2
- Samples with read sums below 10^3 are pruned in the reference workflow, and taxa absent from all samples are filtered out
- R version 4.0.5 and tidyverse 1.3.1 are required dependencies; VTwins is not on CRAN and must be installed from GitHub via remotes

## Evidence

- [abstract] we develop a straightforward method-virtual twins (VTwins)-to eliminate the confounder effects by transforming the original cohort into a paired cohort of "Twin" samples: "we develop a straightforward method-virtual twins (VTwins)-to eliminate the confounder effects by transforming the original cohort into a paired cohort of "Twin" samples"
- [abstract] VTwins outperforms the conventional approach in sensitivity and 10-fold smaller sample size: "The results show that VTwins outperforms the conventional approach in the sensitivity of identifying causative features and only requires a 10-fold reduced sample size"
- [methods] decostand total normalization step: "dataset.normalized <- decostand(dataset,method = "total",1) #normalization for fetures like species's relative abundance"
- [readme] pair_find must keep same order of samples in abundance and phenotype data: "Must keep the `Same Order` of samples in  relative abundance dataframe and phenotype dataframe. And we also keep the samples `clustered and placed` according the `grp1 and grp2`"
- [readme] pair_find returns nothing for fewer than 10 pairs: "**Note:** if the sample pairs are less than 10, it will return nothing."
- [methods] PvalueCutoff parameter description: "PvalueCutoff: p value cutoff of `Incre.aveRank.P` and `Decre.aveRank.P`"
