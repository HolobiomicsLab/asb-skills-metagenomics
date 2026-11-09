---
name: insertion-sequence-candidate-filtering
description: Use when you have FASTA files of candidate ITR and flanking IS-region
  sequences from an upstream maximal-exact-match step and need to call high-confidence
  insertion sequences via ITR cluster concordance and reverse-complement checks.
when_to_use_negative:
- The transposable elements of interest lack inverted terminal repeats — Palidis 'is
  currently only equipped with discovering ISs with ITRs'
- ITRs are shorter than the default 25 nt minimum (as short as 10 nt can be missed
  at default settings)
- Candidate regions are lengthened by nested insertion events so the flanking ITRs
  fall outside the 500–3000 bp proximity range
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Palidis
  role: Pipeline (palidis4/Nextflow) whose getCandidateITRs, clusterReads and getITRs
    steps perform ITR candidate clustering, cluster-concordance filtering and IS output
  repo: https://github.com/blue-moon22/palidis4
- name: CD-HIT-EST
  role: Clusters candidate ITR sequences (G 0, c 0.9, aL 0.0, aS 0.9) so flanking
    ITR pairs can be checked for same-cluster membership
- name: blastn
  role: Verifies paired ITRs are reverse complements using -task blastn -word_size
    4 on plus/minus strands with identities >= minimum ITR length
- name: pal-MEM
  role: Upstream tool producing the repeat/ITR sequences that this filtering step
    consumes
  repo: https://github.com/blue-moon22/pal-MEM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:5b45f001f290811e043475c80dd2bba7522fe70580c8f30a9f20465a21838489
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis4
  tools:
  - CD-HIT-EST
  - Palidis
  - blastn
  - pal-MEM
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# insertion-sequence-candidate-filtering

## Summary

Converts Palidis candidate ITRs into predicted insertion sequences by clustering candidate ITRs with CD-HIT-EST and retaining only candidates whose two flanking ITRs fall in the same cluster and are reverse complements of each other, within IS length bounds.

## When to use

Use when you have FASTA files of candidate ITR and flanking IS-region sequences from an upstream maximal-exact-match step and need to call high-confidence insertion sequences via ITR cluster concordance and reverse-complement checks.

## When NOT to use

- The transposable elements of interest lack inverted terminal repeats — Palidis 'is currently only equipped with discovering ISs with ITRs'
- ITRs are shorter than the default 25 nt minimum (as short as 10 nt can be missed at default settings)
- Candidate regions are lengthened by nested insertion events so the flanking ITRs fall outside the 500–3000 bp proximity range

## Inputs

- FASTA file of candidate ITR sequences from the upstream Palidis matching step
- FASTA file of candidate IS (flanked) region sequences
- IS length bounds and minimum ITR length parameters (defaults: 500–3000 bp, min ITR 25 nt)

## Outputs

- is_candidates.fasta containing IS sequences passing cluster-concordance and reverse-complement checks
- CD-HIT-EST cluster assignments for candidate ITRs

## How to apply

Load the candidate ITR and IS region FASTA files produced by the upstream Palidis matching/filtering steps and cluster the ITR sequences with CD-HIT-EST using global score mode G 0, identity threshold c 0.9, aL 0.0 and aS 0.9. For each candidate IS, verify that its two flanking ITRs map to the same CD-HIT-EST cluster (cluster concordance), which is Palidis' rationale for calling an ITR pair. Apply the IS length bounds (default 500–3000 bp) and minimum ITR length (default 25 nt, max 50 nt) as hard filters. Confirm the paired ITRs are reverse complements by running blastn with -task blastn -word_size 4 restricted to plus/minus strands, requiring identities at least equal to the minimum ITR length. Write all candidates passing both the cluster-concordance and reverse-complement checks to is_candidates.fasta.

## Related tools

- **Palidis** (Pipeline (palidis4/Nextflow) whose getCandidateITRs, clusterReads and getITRs steps perform ITR candidate clustering, cluster-concordance filtering and IS output) — https://github.com/blue-moon22/palidis4
- **CD-HIT-EST** (Clusters candidate ITR sequences (G 0, c 0.9, aL 0.0, aS 0.9) so flanking ITR pairs can be checked for same-cluster membership)
- **blastn** (Verifies paired ITRs are reverse complements using -task blastn -word_size 4 on plus/minus strands with identities >= minimum ITR length)
- **pal-MEM** (Upstream tool producing the repeat/ITR sequences that this filtering step consumes) — https://github.com/blue-moon22/pal-MEM

## Examples

```
cd-hit-est -i candidate_itrs.fasta -o itr_clusters.fasta -G 0 -c 0.9 -aL 0.0 -aS 0.9
```

## Evaluation signals

- Every retained IS candidate has both flanking ITRs assigned to the same CD-HIT-EST cluster
- blastn (word_size 4, plus/minus strands) reports identities >= the minimum ITR length (default 25) for each ITR pair
- All output IS lengths fall within the configured bounds (default 500–3000 bp) and ITR lengths within 25–50 nt
- Palidis4 regression test passes: './tests/regression_tests.sh' prints 'Test passed. All outputs expected.'
- Output files are the expected pair: a FASTA of insertion sequences plus a per-IS information table (IS_name, sample_id, contig, is_start, is_end, description)

## Limitations

- Palidis cannot discover ISs lacking ITRs; the ISC contains only common DDE-type ISs with ITRs
- Default minimum ITR length of 25 nt misses ITRs as short as ten nucleotides; lowering thresholds and k-mer length is more computationally intensive on complex mixed microbial genomes
- ISs flanked by ITRs beyond the 500–3000 bp proximity range (e.g. lengthened by nested insertions) are missed; the maximum IS length may need increasing in future ISC iterations

## Evidence

- [discussion] Palidis is currently only equipped with discovering ISs with ITRs.: "Palidis is currently only equipped with discovering ISs with ITRs."
- [abstract] recognizes insertion sequences in metagenomic sequence data rapidly by identifying inverted terminal repeat regions: "recognizes insertion sequences in metagenomic sequence data rapidly by identifying inverted terminal repeat regions"
- [readme] Cluster candidate ITRs using CD-HIT-EST [`clusterReads`]: "5. Cluster candidate ITRs using CD-HIT-EST [`clusterReads`]
6. Get putative ITRs by cluster concordance and output Insertion Sequences [`getITRs`]"
- [discussion] the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis: "the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis, although ITRs can be as short as ten nucleotides in length"
