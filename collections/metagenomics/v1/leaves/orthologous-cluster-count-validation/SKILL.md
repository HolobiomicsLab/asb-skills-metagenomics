---
name: orthologous-cluster-count-validation
description: Use when you have run (or are about to trust) all-vs-all BLASTP + modified
  Jaccard clustering on the VIRGO non-redundant gene catalog and must verify the resulting
  VOG cluster tables are complete and correctly merged.
when_to_use_negative:
- The task is mapping sample reads against VIRGO for taxonomic/functional profiling
  (use VIRGO Module 1 runMapping.step1.sh/step2.sh instead of Jaccard clustering).
- You only need to BLAST a single sequence or gene list against VIRGO (Module 6 blastn/blastp),
  not to build or validate orthologous protein families.
- Your input is not the VIRGO non-redundant polypeptide set — this skill's expected
  cluster counts are specific to that catalog and cannot be used as invariants for
  other gene catalogs.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: VIRGO
  role: Source non-redundant gene catalog and host of the 4_Jaccard_index_clustering
    module used to generate VOGs
  repo: github.com/ravel-lab/VIRGO
- name: Wu-blastp
  role: All-versus-all BLASTP producing BSML files whose hits are filtered (80% identity,
    70% coverage overlap, E-value < 1e-10) before clustering
- name: clusterBsmlPairwiseAlignments-mod.pl
  role: Computes modified Jaccard similarity clusters from the filtered polypeptide
    pair list with linkscore=0.6
- name: CogBsmlLoader-mod.pl
  role: Converts Wu-blastp BSML results to a best-hit btab summary embedding Jaccard
    cluster info in the last column
- name: best_hit.pl
  role: Merges all Jaccard clusters with reciprocal best hits to produce final-clusters.cog.gz
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:a1f84ddcd9941f3dc96593a02c1d6f749a64f40d11880c9871c2948928ae36bd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CogBsmlLoader-mod.pl
  - VIRGO
  - Wu-blastp
  - best_hit.pl
  - clusterBsmlPairwiseAlignments-mod.pl
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# orthologous-cluster-count-validation

## Summary

Validates the output of Jaccard-index clustering of the VIRGO non-redundant polypeptide set into Vaginal Orthologous Groups (VOGs), checking that the produced JAC/JOC cluster counts, multi-gene clusters, and singleton fractions match expected invariants.

## When to use

Use when you have run (or are about to trust) all-vs-all BLASTP + modified Jaccard clustering on the VIRGO non-redundant gene catalog and must verify the resulting VOG cluster tables are complete and correctly merged.

## When NOT to use

- The task is mapping sample reads against VIRGO for taxonomic/functional profiling (use VIRGO Module 1 runMapping.step1.sh/step2.sh instead of Jaccard clustering).
- You only need to BLAST a single sequence or gene list against VIRGO (Module 6 blastn/blastp), not to build or validate orthologous protein families.
- Your input is not the VIRGO non-redundant polypeptide set — this skill's expected cluster counts are specific to that catalog and cannot be used as invariants for other gene catalogs.

## Inputs

- VIRGO non-redundant polypeptide (protein) FASTA set (~0.95 million non-redundant genes)
- Wu-blastp BSML output files referenced by a Wu-blastp.bsml.list file
- Filtered polypeptide pair list (e.g. 20808.pairs.gz) of BLASTP hits passing the identity/coverage/E-value filters

## Outputs

- Jaccard-clusters.out: JAC clusters from clusterBsmlPairwiseAlignments-mod.pl
- All-best-hits.btab: best-hit summary with Jaccard cluster info in the last column
- final-clusters.cog.gz: final VOG cluster file after reciprocal-best-hit merging (617,127 JACs; 552,679 JOCs; 177,684 multi-gene clusters; 374,995 singletons)

## How to apply

Run all-versus-all Wu-blastp on the VIRGO non-redundant polypeptides, filtering hits at 80% identity, 70% coverage overlap, and E-value < 1e-10, then compute modified Jaccard clusters with clusterBsmlPairwiseAlignments-mod.pl at linkscore=0.6 (optionally pruning with a minimum Jaccard coefficient). Convert the Wu-blastp BSML results to a best-hit btab summary with CogBsmlLoader-mod.pl, carrying Jaccard cluster IDs in the last column, then merge clusters with reciprocal best hits using best_hit.pl to produce final-clusters.cog.gz. Validate by checking the cluster tables against the expected decomposition: 617,127 JACs and 552,679 JOCs, of which 177,684 contain at least two genes and 374,995 are singletons (~38.5% of VOG proteins unique). The 177,684 multi-gene + 374,995 singleton counts must sum exactly to the JOC total; any discrepancy indicates a broken merge or parsing step.

## Related tools

- **VIRGO** (Source non-redundant gene catalog and host of the 4_Jaccard_index_clustering module used to generate VOGs) — github.com/ravel-lab/VIRGO
- **Wu-blastp** (All-versus-all BLASTP producing BSML files whose hits are filtered (80% identity, 70% coverage overlap, E-value < 1e-10) before clustering)
- **clusterBsmlPairwiseAlignments-mod.pl** (Computes modified Jaccard similarity clusters from the filtered polypeptide pair list with linkscore=0.6)
- **CogBsmlLoader-mod.pl** (Converts Wu-blastp BSML results to a best-hit btab summary embedding Jaccard cluster info in the last column)
- **best_hit.pl** (Merges all Jaccard clusters with reciprocal best hits to produce final-clusters.cog.gz)

## Examples

```
./run-jaccard.sh
```

## Evaluation signals

- Jaccard-clusters.out contains 617,127 JACs
- Final cluster file contains 552,679 JOCs, of which 177,684 are multi-gene clusters and 374,995 are singletons (counts must sum to the JOC total)
- Singleton fraction equals ~38.5% of VOG proteins
- All-best-hits.btab rows carry a valid Jaccard cluster identifier in the last column
- final-clusters.cog.gz exists and is non-empty after best_hit.pl reciprocal-best-hit merging

## Limitations

- Clustering parameters are tunable (linkscore, percent_identity=80, percent_coverage=60, p_value=1e-5) and a minimum Jaccard coefficient can be set for extra cluster pruning, so deviations from the reference counts are expected if thresholds are changed.
- VIRGO's database is primarily bacterial with limited inclusion of viral and fungal gene sequences, so VOG coverage of non-bacterial proteins is limited.
- The reference-based approach has acknowledged limitations; this VIRGO version focuses on gene-level de-redundancy of the vaginal microbiome.

## Evidence

- [results] Clustering the VIRGO non-redundant genes generated 617,127 JACs and 552,679 JOCs, of which 177,684 contained at least two genes and 374,995 were singletons (38.5% of VOG proteins are unique).: [evidence span withheld — anchor status: not_found]
- [abstract] we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes: "we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes"
- [readme] ### MODULE 7. Using jaccard clustering to generate protein cluster###: "### MODULE 7. Using jaccard clustering to generate protein cluster###"
- [methods] Run all-versus-all BLASTP on the VIRGO non-redundant polypeptide set with Wu-blastp, filtering hits at 80% percent identity, 70% percent coverage overlap, and E-value < 1e-10.: [evidence span withheld — anchor status: not_found]
- [methods] Minimum Jaccard coefficient (for additional cluster pruning if set to > 0): "Minimum Jaccard coefficient (for additional cluster pruning if set to > 0)"
