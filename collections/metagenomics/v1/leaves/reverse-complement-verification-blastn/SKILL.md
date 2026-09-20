---
name: reverse-complement-verification-blastn
description: Use when converting Palidis candidate ITR pairs into predicted insertion
  sequences and you need cluster-concordance plus reverse-complement evidence that
  the flanking repeat pair is a genuine ITR.
when_to_use_negative:
- ISs lacking ITRs — Palidis is only equipped with discovering ISs with ITRs, so cluster
  concordance cannot be evaluated
- Candidate pairs whose ITRs fall outside the 500–3000 bp proximity range (e.g. lengthened
  by nested insertion events) will be missed regardless of clustering
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CD-HIT-EST
  role: Clusters candidate ITR sequences so that pairs sharing a cluster provide concordance
    evidence for ITR identity
- name: blastn
  role: Verifies paired ITRs are reverse complements (task blastn, word_size 4, plus/minus
    strands, identities >= min ITR length)
- name: Palidis
  role: Upstream pipeline producing the candidate ITR and IS region sequences; this
    skill corresponds to its clusterReads/getITRs steps
  repo: https://github.com/blue-moon22/palidis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:748b213d801da108592ba1e98d3894409e88ef28e23719fa4f7e66469829125a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis
  tools:
  - CD-HIT-EST
  - Palidis
  - blastn
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# reverse-complement-verification-blastn

## Summary

Palidis Step 3 converts candidate ITR pairs into predicted insertion sequences by clustering candidate ITR sequences with CD-HIT-EST and retaining only candidates whose two flanking ITRs fall in the same cluster and are reverse complements of each other, verified with blastn.

## When to use

Use when converting Palidis candidate ITR pairs into predicted insertion sequences and you need cluster-concordance plus reverse-complement evidence that the flanking repeat pair is a genuine ITR.

## When NOT to use

- ISs lacking ITRs — Palidis is only equipped with discovering ISs with ITRs, so cluster concordance cannot be evaluated
- Candidate pairs whose ITRs fall outside the 500–3000 bp proximity range (e.g. lengthened by nested insertion events) will be missed regardless of clustering

## Inputs

- Candidate ITR sequences (FASTA) from the upstream Palidis matching step
- Candidate IS region sequences (FASTA) with flanking ITR pairs
- IS length bounds (default 500–3000 bp) and minimum ITR length (default 25 nt)

## Outputs

- is_candidates.fasta containing IS candidates passing cluster-concordance and reverse-complement checks
- CD-HIT-EST cluster assignments of candidate ITRs

## How to apply

Load the candidate ITR and IS-region FASTA sequences produced by the upstream Palidis matching step and cluster the candidate ITR sequences with CD-HIT-EST using global score mode G 0, identity threshold c 0.9, aL 0.0 and aS 0.9. For each candidate insertion sequence, check that its two flanking ITRs fall within the same CD-HIT-EST cluster; only keep candidates whose total length lies within the IS length bounds (default 500–3000 bp) and whose ITRs meet the minimum ITR length (default 25 nt). Verify that the paired ITRs are reverse complements by running blastn with -task blastn -word_size 4, restricting to plus/minus strands, and requiring the number of identities to be at least the minimum ITR length. Write all candidates passing both the cluster-concordance and reverse-complement checks to is_candidates.fasta.

## Related tools

- **CD-HIT-EST** (Clusters candidate ITR sequences so that pairs sharing a cluster provide concordance evidence for ITR identity)
- **blastn** (Verifies paired ITRs are reverse complements (task blastn, word_size 4, plus/minus strands, identities >= min ITR length))
- **Palidis** (Upstream pipeline producing the candidate ITR and IS region sequences; this skill corresponds to its clusterReads/getITRs steps) — https://github.com/blue-moon22/palidis

## Examples

```
cd-hit-est -i candidate_itrs.fasta -o itr_clusters -G 0 -c 0.9 -aL 0.0 -aS 0.9
```

## Evaluation signals

- Every retained IS has both flanking ITRs assigned to the same CD-HIT-EST cluster (c 0.9, aS 0.9)
- For each retained ITR pair, the blastn alignment (word_size 4, plus/minus) reports identities >= the minimum ITR length (default 25)
- All output IS lengths fall within the configured bounds (default 500–3000 bp) and ITR lengths within min/max ITR thresholds (default 25–50 nt)
- Output is a valid FASTA file (is_candidates.fasta) whose entries trace back to candidate pairs from the upstream matching step

## Limitations

- Default minimum ITR length of 25 nt means ITRs as short as ten nucleotides are missed; lowering the threshold and k-mer length is more computationally intensive
- ISs flanked by ITRs beyond the 500–3000 bp proximity range (e.g. regions lengthened by nested insertion events) are missed
- Only common DDE-type ISs with ITRs are captured; ISs without ITRs cannot be discovered

## Evidence

- [readme] Cluster candidate ITRs using CD-HIT-EST [`clusterReads`]: "5. Cluster candidate ITRs using CD-HIT-EST [`clusterReads`]"
- [readme] Get putative ITRs by cluster concordance and output Insertion Sequences: "6. Get putative ITRs by cluster concordance and output Insertion Sequences [`getITRs`]"
- [intro] Palidis finds ISs using an efficient maximal exact matching algorithm: "we present a tool called Palidis (palindromic detection of insertion sequences) that finds ISs using an efficient maximal exact matching algorithm"
- [discussion] ITRs 25 or greater nucleotides; ITRs can be as short as ten nucleotides: "the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis, although ITRs can be as short as ten nucleotides in length"
