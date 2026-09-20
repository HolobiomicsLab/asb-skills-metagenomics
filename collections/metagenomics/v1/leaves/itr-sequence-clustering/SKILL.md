---
name: itr-sequence-clustering
description: Use when you have FASTA files of candidate ITR and IS-region sequences
  from an upstream Palidis maximal-exact-match step and need to filter them into high-confidence
  IS predictions via cluster concordance and reverse-complement checks.
when_to_use_negative:
- Input ISs lack ITRs entirely — Palidis is 'only equipped with discovering ISs with
  ITRs', so cluster-concordance filtering cannot apply
- Candidate ITRs are shorter than the minimum ITR length threshold (default 25 nt;
  ITRs as short as 10 nt are missed at default settings)
- Candidate IS regions exceed the 500–3000 bp proximity range (e.g. lengthened by
  nested insertion events) and will be filtered out regardless of clustering
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Palidis
  role: Upstream pipeline that produces the candidate ITR and IS region sequences
    being clustered; Step 5 'clusterReads' and Step 6 'getITRs' implement this skill
  repo: https://github.com/blue-moon22/palidis
- name: CD-HIT-EST
  role: Clusters candidate ITR nucleotide sequences with G 0, c 0.9, aL 0.0, aS 0.9
    to define ITR families
- name: blastn
  role: Verifies paired ITRs are reverse complements using -task blastn -word_size
    4 on plus/minus strands, requiring identities >= minimum ITR length
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:a8e13e979ad317d6e6e1f9e11fe471eca17ad0f141db4d27391f8a738af3b1c8
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# itr-sequence-clustering

## Summary

Cluster candidate inverted terminal repeat (ITR) sequences from the Palidis pipeline using CD-HIT-EST, then convert candidates into predicted insertion sequences by retaining candidates whose paired ITRs fall in the same cluster and are reverse complements of each other (verified with blastn).

## When to use

Use when you have FASTA files of candidate ITR and IS-region sequences from an upstream Palidis maximal-exact-match step and need to filter them into high-confidence IS predictions via cluster concordance and reverse-complement checks.

## When NOT to use

- Input ISs lack ITRs entirely — Palidis is 'only equipped with discovering ISs with ITRs', so cluster-concordance filtering cannot apply
- Candidate ITRs are shorter than the minimum ITR length threshold (default 25 nt; ITRs as short as 10 nt are missed at default settings)
- Candidate IS regions exceed the 500–3000 bp proximity range (e.g. lengthened by nested insertion events) and will be filtered out regardless of clustering

## Inputs

- FASTA file of candidate ITR sequences (output of upstream Palidis matching/getCandidateITRs step)
- FASTA file of candidate IS region sequences (output of upstream Palidis matching step)
- IS length bounds (default min 500 bp, max 3000 bp) and minimum ITR length threshold (default 25 nt)

## Outputs

- CD-HIT-EST cluster assignments for candidate ITR sequences
- is_candidates.fasta containing IS candidates passing cluster-concordance and reverse-complement checks

## How to apply

Load the candidate ITR and IS region FASTA files produced by the upstream Palidis matching step and cluster the candidate ITRs with CD-HIT-EST using the parameters: global score mode G 0, sequence identity threshold c 0.9, alignment coverage for the longer sequence aL 0.0, and alignment coverage for the shorter sequence aS 0.9. For each candidate IS, check that its two flanking ITRs fall within the same CD-HIT-EST cluster (cluster concordance). Retain only candidates whose total length falls within the IS length bounds (default 500–3000 bp per the palidis4 README) and whose ITRs meet the minimum ITR length requirement (default 25 nt). Verify that paired ITRs are reverse complements by running blastn with -task blastn -word_size 4, restricting to plus/minus strands, and requiring the number of identities to be at least the minimum ITR length. Write all IS candidates passing both the cluster-concordance and reverse-complement checks to is_candidates.fasta.

## Related tools

- **Palidis** (Upstream pipeline that produces the candidate ITR and IS region sequences being clustered; Step 5 'clusterReads' and Step 6 'getITRs' implement this skill) — https://github.com/blue-moon22/palidis
- **CD-HIT-EST** (Clusters candidate ITR nucleotide sequences with G 0, c 0.9, aL 0.0, aS 0.9 to define ITR families)
- **blastn** (Verifies paired ITRs are reverse complements using -task blastn -word_size 4 on plus/minus strands, requiring identities >= minimum ITR length)

## Examples

```
cd-hit-est -i candidate_itrs.fasta -o itr_clusters -G 0 -c 0.9 -aL 0.0 -aS 0.9
```

## Evaluation signals

- Every retained IS candidate has both flanking ITRs assigned to the same CD-HIT-EST cluster
- blastn alignment between paired ITRs reports number of identities >= the minimum ITR length on opposite (plus/minus) strands
- All output IS candidates fall within the IS length bounds (default 500–3000 bp) and have ITRs at least the minimum ITR length (default 25 nt)
- is_candidates.fasta is non-empty and contains only candidates that passed both concordance and reverse-complement checks
- palidis4 regression test passes: './tests/regression_tests.sh' prints 'Test passed. All outputs expected.'

## Limitations

- ISC/Palidis only captures common DDE-type ISs with ITRs; ISs lacking ITRs are not discoverable
- ITRs as short as ten nucleotides are missed at default settings (minimum ITR length default 25), and lowering the threshold with smaller k-mer length is more computationally intensive
- ISs flanked by ITRs beyond the 500–3000 bp proximity range (e.g. lengthened by nested insertion events) are missed; increasing maximum IS length may be needed in future iterations

## Evidence

- [other] Palidis Step 3 clusters candidate ITRs using CD-HIT-EST and identifies insertion sequences by ITRs that belong to the same cluster and are reverse complements of each other.: [evidence span withheld — anchor status: not_found]
- [readme] README steps 5-6 of palidis4 pipeline: "5. Cluster candidate ITRs using CD-HIT-EST [`clusterReads`]
6. Get putative ITRs by cluster concordance and output Insertion Sequences [`getITRs`]"
- [other] CD-HIT-EST parameters from workflow: [evidence span withheld — anchor status: not_found]
- [discussion] Palidis limitation on ISs with ITRs: "Palidis is currently only equipped with discovering ISs with ITRs."
- [readme] Minimum ITR length default: "--min_itr_length    Minimum length of ITR. (Default: 25)"
