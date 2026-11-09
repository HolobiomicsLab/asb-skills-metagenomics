---
name: protein-cluster-consistency-scoring
description: Use when you have clusterings of a large protein sequence set (e.g.,
  UniProt) produced by multiple tools at several identity thresholds and need to quantify
  which tool yields functionally most consistent clusters using GO/Pfam annotations.
when_to_use_negative:
- When the goal is to produce a redundancy-reduced sequence set rather than evaluate
  clustering quality — run a clustering tool directly instead
- When the input clusterings lack a designated representative sequence per cluster,
  since both 'worst' and 'mean' scores and the Pfam check are computed against the
  representative
- When annotations are too sparse for meaningful pairwise annotation similarity (no
  GO/Pfam terms for most members)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: Linclust
  role: Linear-time clustering tool whose clusterings at 90/70/50% identity are scored
    for GO/Pfam annotation consistency; invoked via MMseqs2 linclust
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Suite in which Linclust is integrated; provides the linclust clustering commands
    used to generate the evaluated clusterings
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Baseline clustering tool whose clusters are scored for annotation consistency
    as the comparison standard
- name: UCLUST 7.0.1090
  role: Baseline clustering tool whose clusters are scored for annotation consistency
    as the comparison standard
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:efa166dfcfea147331e381a59868d4923752d375eb11dbf19b3e58d77e544bd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - CD-HIT 4.6
  - Linclust
  - MMseqs2
  - UCLUST
  - UCLUST 7.0.1090
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# protein-cluster-consistency-scoring

## Summary

Score the functional consistency of protein sequence clusterings by comparing each cluster representative's Gene Ontology and Pfam annotations against its members, using 'mean' and 'worst' per-cluster similarity scores. This validates whether faster clusterers (e.g., Linclust) preserve annotation quality relative to established tools (UCLUST, CD-HIT).

## When to use

Use when you have clusterings of a large protein sequence set (e.g., UniProt) produced by multiple tools at several identity thresholds and need to quantify which tool yields functionally most consistent clusters using GO/Pfam annotations.

## When NOT to use

- When the goal is to produce a redundancy-reduced sequence set rather than evaluate clustering quality — run a clustering tool directly instead
- When the input clusterings lack a designated representative sequence per cluster, since both 'worst' and 'mean' scores and the Pfam check are computed against the representative
- When annotations are too sparse for meaningful pairwise annotation similarity (no GO/Pfam terms for most members)

## Inputs

- Protein sequence set clustered by multiple tools (UniProt 2016_03 release clusterings from Linclust, Linclust/MMseqs2 commit 5e21868, UCLUST 7.0.1090, CD-HIT 4.6) at 90%, 70%, and 50% sequence identity
- UniProt knowledge base Gene Ontology annotations, split into experimentally validated and general (homology-inferred) subsets
- UniProt Pfam domain annotations

## Outputs

- cluster_consistency_scores.csv with 'mean' and 'worst' GO consistency score columns for each tool/identity-threshold combination
- Per-cluster Pfam local consistency counts (exact domain-annotation match = 1, mismatch = 0) aggregated per tool and identity level

## How to apply

Cluster the sequence set (e.g., UniProt 2016_03 release) with each tool at fixed sequence-identity levels — here 90%, 70%, and 50% — using the exact published parameters: Linclust via `mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9` and CD-HIT 4.6 with `-T 16 -M 0 -n 5 -c 0.9` (and `-n 4 -c 0.7`, `-n 3 -c 0.5`). Parse UniProt knowledge base GO annotations, separating experimentally validated GO terms from general functional GO terms mostly inferred from homologous proteins. For each cluster, compute the 'worst' consistency score as the minimum and the 'mean' consistency score as the mean of all pairwise annotation-similarity scores between the representative sequence and the other cluster members. Separately check local consistency with Pfam annotations of UniProt, counting exact domain-annotation matches as correct (1) and mismatches as false (0) against the representative. Aggregate per-cluster scores per tool and identity level into a single table (cluster_consistency_scores.csv) and compare tools across identity levels.

## Related tools

- **Linclust** (Linear-time clustering tool whose clusterings at 90/70/50% identity are scored for GO/Pfam annotation consistency; invoked via MMseqs2 linclust) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Suite in which Linclust is integrated; provides the linclust clustering commands used to generate the evaluated clusterings) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Baseline clustering tool whose clusters are scored for annotation consistency as the comparison standard)
- **UCLUST 7.0.1090** (Baseline clustering tool whose clusters are scored for annotation consistency as the comparison standard)

## Examples

```
mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9
```

## Evaluation signals

- cluster_consistency_scores.csv exists and contains 'mean' and 'worst' consistency score columns for each tool/identity-threshold combination
- All consistency scores are bounded in [0, 1], as expected for annotation similarity scores
- Pfam local consistency counts are binary (exact domain-annotation match = 1, mismatch = 0) per representative–member pair
- Scores are aggregated per tool and per sequence-identity level, allowing direct Linclust vs. UCLUST/CD-HIT comparison at matching identity levels
- Expected outcome: Linclust/MMseqs2 and Linclust show better annotation consistency than UCLUST and CD-HIT for experimentally derived GO annotations and for Pfam domain annotations

## Limitations

- Linclust's k-mer prefilter-based clustering is approximate; if falsely clustered pairs are costly (e.g., for redundancy filtering), pairwise links must be verified with the exact similarity measure, which still requires quadratic time
- General functional GO annotations are mostly inferred from homologous proteins, so they are partially self-referential for clustering quality — experimentally validated GO annotations are the stricter criterion
- Clustering quality is assessed only at fixed identity thresholds (90%, 70%, 50%) and against the tools' default representative-choice conventions (greedy incremental clustering picks the longest sequence as representative)

## Evidence

- [methods] GO annotation separation: "We evaluated the functional cluster consistency based on Gene Ontology (GO) annotations of the UniProt knowledge base."
- [methods] Pfam local consistency: "We used the Pfam annotations of the UniProt to check local consistence of clusters (Fig. 3c)."
- [methods] Identity levels evaluated: "The UniProt 2016_03 release was clustered by each tool at 90%, 70% and 50% sequence identity level and then evaluated."
- [methods] Linclust command: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] CD-HIT parameters: "CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5"
- [discussion] Prefilter approximation caveat: "If falsely clustered pairs are costly (e.g., for redundancy filtering), pairwise links need to be verified with the exact similarity measure, which still requires quadratic time complexity."
