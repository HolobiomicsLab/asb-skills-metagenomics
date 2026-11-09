---
name: gene-ontology-annotation-similarity-computation
description: Use when you need to assess the functional coherence of protein sequence
  clusters (e.g., produced by Linclust, UCLUST, CD-HIT) via GO or Pfam annotation
  agreement.
when_to_use_negative:
- When only a clustering without its representative-sequence annotations is available
  — the score is computed against the cluster's representative sequence
- When the goal is measuring clustering speed or memory footprint rather than annotation
  consistency
- When annotations are not structured as GO terms or Pfam domains, since the similarity
  measure relies on exact domain-annotation matching
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_0623
tools:
- name: MMseqs2
  role: Software suite whose linclust command produced the clusterings evaluated for
    annotation consistency
  repo: https://github.com/soedinglab/mmseqs2
- name: Linclust
  role: Linear-time clustering tool whose clusters are scored for GO/Pfam annotation
    consistency
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Comparison clustering tool (run with -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7,
    -n 3 -c 0.5) whose cluster consistency is scored
- name: UCLUST
  role: Comparison clustering tool (7.0.1090) whose cluster consistency is scored
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:4a54ed7f7a71b7d6e4cb70ff62febeefd2313056bdaab6f2208b42bc80ae02c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - CD-HIT 4.6
  - Linclust
  - MMseqs2
  - UCLUST
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gene-ontology-annotation-similarity-computation

## Summary

A method for scoring cluster annotation consistency using Gene Ontology and Pfam annotations from the UniProt knowledge base, comparing each cluster's representative sequence against all cluster members with 'mean' and 'worst' per-cluster similarity scores.

## When to use

Use when you need to assess the functional coherence of protein sequence clusters (e.g., produced by Linclust, UCLUST, CD-HIT) via GO or Pfam annotation agreement.

## When NOT to use

- When only a clustering without its representative-sequence annotations is available — the score is computed against the cluster's representative sequence
- When the goal is measuring clustering speed or memory footprint rather than annotation consistency
- When annotations are not structured as GO terms or Pfam domains, since the similarity measure relies on exact domain-annotation matching

## Inputs

- Clusterings of the UniProt 2016_03 release at 90%, 70%, and 50% sequence identity from Linclust, Linclust/MMseqs2 (commit 5e21868), UCLUST 7.0.1090, and CD-HIT 4.6
- Gene Ontology annotations from the UniProt knowledge base (experimentally validated and general/homology-inferred)
- Pfam annotations from UniProt

## Outputs

- cluster_consistency_scores.csv with 'mean' and 'worst' consistency score columns per tool/identity-threshold combination
- Per-cluster pairwise annotation similarity scores bounded in [0, 1]
- Pfam local consistency counts (exact domain-annotation match = 1, mismatch = 0)

## How to apply

Load the clusterings (e.g., of UniProt 2016_03 at 90%, 70%, and 50% sequence identity from Linclust, Linclust/MMseqs2, UCLUST 7.0.1090, and CD-HIT 4.6) and parse the GO annotations from the UniProt knowledge base, separating experimentally validated GO annotations from general functional GO annotations mostly inferred from homologous proteins. For each cluster, compute the 'worst' consistency score as the minimum and the 'mean' consistency score as the mean of all pairwise annotation similarity scores between the cluster's representative sequence and the other cluster members. For local consistency, compare the Pfam domain annotation of each representative sequence against all cluster members, counting exact domain-annotation matches as correct (1) and mismatches as false (0). Aggregate per-cluster scores per tool and per sequence-identity level into a CSV such as cluster_consistency_scores.csv, with all similarity scores bounded in [0, 1].

## Related tools

- **MMseqs2** (Software suite whose linclust command produced the clusterings evaluated for annotation consistency) — https://github.com/soedinglab/mmseqs2
- **Linclust** (Linear-time clustering tool whose clusters are scored for GO/Pfam annotation consistency) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Comparison clustering tool (run with -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, -n 3 -c 0.5) whose cluster consistency is scored)
- **UCLUST** (Comparison clustering tool (7.0.1090) whose cluster consistency is scored)

## Examples

```
mmseqs linclust uniprot2016_03 clustering_tmp clustering --cov-mode 1 -c 0.9 --min-seq-id 0.9; python compute_consistency.py --clusters clustering --uniprot uniprot_sprot.dat --out cluster_consistency_scores.csv
```

## Evaluation signals

- Output CSV cluster_consistency_scores.csv exists with 'mean' and 'worst' consistency score columns for each tool/identity-threshold combination
- All annotation similarity scores are bounded in [0, 1]
- Pfam local consistency is binary: exact domain-annotation matches counted as 1, mismatches as 0
- Scores are aggregated per clustering tool (Linclust, Linclust/MMseqs2, UCLUST, CD-HIT) and per sequence-identity level (90%, 70%, 50%)
- Experimentally validated GO annotations are analyzed separately from general functional GO annotations

## Limitations

- General functional GO annotations are mostly inferred from homologous proteins, which can inflate apparent consistency; purely experimentally derived GO annotations should be analyzed separately
- The 'worst' score is a minimum over cluster members, so it is sensitive to a single outlier sequence in a large cluster
- Pfam-based local consistency treats annotation comparison as exact match or mismatch, giving no partial credit

## Evidence

- [methods] We evaluated the functional cluster consistency based on Gene Ontology (GO) annotations of the UniProt knowledge base.: "We evaluated the functional cluster consistency based on Gene Ontology (GO) annotations of the UniProt knowledge base."
- [methods] We used the Pfam annotations of the UniProt to check local consistence of clusters (Fig. 3c).: "We used the Pfam annotations of the UniProt to check local consistence of clusters (Fig. 3c)."
- [methods] The UniProt 2016_03 release was clustered by each tool at 90%, 70% and 50% sequence identity level and then evaluated.: "The UniProt 2016_03 release was clustered by each tool at 90%, 70% and 50% sequence identity level and then evaluated."
- [methods] Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [full_text] Finding: Linclust/MMseqs2 and Linclust clusterings show better annotation consistency than UCLUST and CD-HIT.: [evidence span withheld — anchor status: not_found]
