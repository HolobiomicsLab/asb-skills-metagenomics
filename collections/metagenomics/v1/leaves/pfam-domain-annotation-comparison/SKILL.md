---
name: pfam-domain-annotation-comparison
description: Use when evaluating the quality of protein sequence clusterings (e.g.
  from Linclust, UCLUST, CD-HIT at 90/70/50% identity) by measuring how often cluster
  members share identical Pfam domain annotations with their cluster representative.
when_to_use_negative:
- When the goal is global functional coherence based on GO terms rather than local
  domain architecture — use the GO-based 'mean'/'worst' pairwise annotation similarity
  scores instead.
- When sequences carry no UniProt Pfam annotations (e.g. novel metagenomic contigs
  without Pfam hits) — the match/mismatch scoring requires existing domain annotations.
- When clustering quality must be judged on structural or phylogenetic criteria rather
  than shared Pfam domain composition.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0080
tools:
- name: Linclust
  role: Produces the clustering whose Pfam domain-annotation local consistency is
    being evaluated; run e.g. as 'mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id
    0.9' and integrated into MMseqs2
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Comparison clustering tool run with -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7,
    -n 3 -c 0.5 to benchmark annotation consistency
- name: UCLUST
  role: Comparison clustering tool (UCLUST 7.0.1090) used as baseline whose clusterings'
    Pfam consistency is scored
- name: MMseqs2
  role: Software suite containing Linclust; source of the Linclust/MMseqs2 clusterings
    evaluated at commit 5e21868
  repo: https://github.com/soedinglab/mmseqs2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:a74453db9e3c351c877bcf37828cfd00cdb4eee0e02749f20aaab6720975720e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - CD-HIT 4.6
  - Linclust
  - MMseqs2
  - UCLUST
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# pfam-domain-annotation-comparison

## Summary

Assesses local functional consistency of protein sequence clusters by comparing the Pfam domain annotations of each cluster representative against all cluster members, scoring exact domain-annotation matches as 1 and mismatches as 0. Used in the Linclust paper to show that Linclust/MMseqs2 clusterings of UniProt 2016_03 are more locally consistent than UCLUST and CD-HIT.

## When to use

Use when evaluating the quality of protein sequence clusterings (e.g. from Linclust, UCLUST, CD-HIT at 90/70/50% identity) by measuring how often cluster members share identical Pfam domain annotations with their cluster representative.

## When NOT to use

- When the goal is global functional coherence based on GO terms rather than local domain architecture — use the GO-based 'mean'/'worst' pairwise annotation similarity scores instead.
- When sequences carry no UniProt Pfam annotations (e.g. novel metagenomic contigs without Pfam hits) — the match/mismatch scoring requires existing domain annotations.
- When clustering quality must be judged on structural or phylogenetic criteria rather than shared Pfam domain composition.

## Inputs

- Protein sequence clusterings (e.g. of the UniProt 2016_03 release) produced by tools such as Linclust, Linclust/MMseqs2 (commit 5e21868), UCLUST 7.0.1090, or CD-HIT 4.6 at 90%, 70% and 50% sequence identity
- Pfam annotations of the UniProt knowledge base, keyed by UniProt accession

## Outputs

- Per-cluster Pfam local-consistency scores (correct=1 / false=0 per member-vs-representative domain-annotation comparison)
- Aggregated per-tool, per-identity-level consistency statistics written to a results file such as cluster_consistency_scores.csv

## How to apply

Take a clustering (e.g. produced by Linclust, Linclust/MMseqs2, UCLUST, or CD-HIT over the UniProt 2016_03 release at 90%, 70%, and 50% sequence identity thresholds) and parse the Pfam annotations of the UniProt knowledge base for every member sequence. For each cluster, compare the Pfam domain annotation of the representative sequence against all cluster members, counting an exact domain-annotation match as correct (1) and a mismatch as false (0). Aggregate these per-cluster local-consistency scores per tool and per sequence-identity level so clusterings can be ranked (e.g. alongside GO-based 'mean' and 'worst' consistency scores, which complement the local Pfam measure). Rationale: Pfam domain architecture is a local, well-annotated functional signal, so clusters whose members agree with the representative's domain composition indicate biologically coherent grouping.

## Related tools

- **Linclust** (Produces the clustering whose Pfam domain-annotation local consistency is being evaluated; run e.g. as 'mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9' and integrated into MMseqs2) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Comparison clustering tool run with -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, -n 3 -c 0.5 to benchmark annotation consistency)
- **UCLUST** (Comparison clustering tool (UCLUST 7.0.1090) used as baseline whose clusterings' Pfam consistency is scored)
- **MMseqs2** (Software suite containing Linclust; source of the Linclust/MMseqs2 clusterings evaluated at commit 5e21868) — https://github.com/soedinglab/mmseqs2

## Examples

```
mmseqs linclust uniprot2016_03 clustering_90 --cov-mode 1 -c 0.9 --min-seq-id 0.9
```

## Evaluation signals

- Output CSV (cluster_consistency_scores.csv) exists with 'mean' and 'worst' consistency score columns for each tool/identity-threshold combination, with all scores bounded in [0, 1].
- Every cluster member's Pfam annotation was compared to exactly one representative, so the number of scored pairs equals total cluster members minus number of clusters.
- Per-member score is binary (0 or 1); any non-binary or out-of-range value indicates a parsing error.
- Aggregated Pfam local consistency for Linclust/Linclust-MMseqs2 clusterings should match or exceed that of UCLUST and CD-HIT at the same identity thresholds (expected finding).
- Scores are computed separately for each identity level (90%, 70%, 50%) so trends across thresholds are preserved.

## Limitations

- The measure is purely annotation-based: sequences lacking Pfam annotations contribute ambiguous match/mismatch outcomes.
- General GO annotations are mostly inferred from homologous proteins, which is why the paper separates experimentally validated GO annotations — an analogous circularity caution applies when domain annotations themselves derive from homology.
- Linclust clustering is approximate; if falsely clustered pairs are costly, pairwise links need verification with an exact similarity measure.
- Pfam comparison captures local (domain) consistency only and does not certify full-length protein function agreement.

## Evidence

- [methods] We used the Pfam annotations of the UniProt to check local consistence of clusters (Fig. 3c).: "We used the Pfam annotations of the UniProt to check local consistence of clusters (Fig. 3c)."
- [methods] The UniProt 2016_03 release was clustered by each tool at 90%, 70% and 50% sequence identity level and then evaluated.: "The UniProt 2016_03 release was clustered by each tool at 90%, 70% and 50% sequence identity level and then evaluated."
- [methods] CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5: "CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5"
- [methods] Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [full_text] Linclust/MMseqs2 and Linclust clusterings show better annotation consistency than UCLUST and CD-HIT for purely experimentally derived GO annotations and for Pfam domain annotations.: [evidence span withheld — anchor status: not_found]
