---
name: short-sequence-nucleotide-alignment
description: Use when two short nucleotide sequences (e.g. candidate inverted terminal
  repeats ≥25 nt, up to 50 nt) must be tested for reverse-complement concordance before
  accepting an IS candidate.
when_to_use_negative:
- The insertion sequences of interest lack ITRs — Palidis is 'currently only equipped
  with discovering ISs with ITRs', so reverse-complement ITR verification is inapplicable
- ITRs are shorter than the minimum ITR length threshold (e.g. 25 nt default; ITRs
  as short as 10 nt are missed at default settings)
- Sequences are long genomic contigs requiring general-purpose homology search rather
  than short paired-sequence concordance testing
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0798
tools:
- name: blastn
  role: Aligns paired candidate ITRs on plus/minus strands with -task blastn -word_size
    4 to verify they are reverse complements; requires identities ≥ minimum ITR length
- name: CD-HIT-EST
  role: Clusters candidate ITR sequences (c 0.9, aS 0.9, aL 0.0, G 0) upstream of
    the alignment check so that both ITRs of a candidate IS must fall in the same
    cluster
- name: Palidis
  role: Pipeline whose Step 3 (clusterReads + getITRs) orchestrates the clustering,
    length filtering, and blastn-based reverse-complement concordance check
  repo: https://github.com/blue-moon22/palidis4
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:a381413fae0da94eb0e300f00cee1c2951b68941fa64c090cc832f003f3baf46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis4
  tools:
  - CD-HIT-EST
  - Palidis
  - blastn
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0798
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# short-sequence-nucleotide-alignment

## Summary

Verifies that paired candidate ITRs are reverse complements of each other using blastn in short-sequence mode (-task blastn -word_size 4), a key concordance check within the Palidis Step 3 conversion of clustered candidate ITRs into predicted insertion sequences.

## When to use

Use when two short nucleotide sequences (e.g. candidate inverted terminal repeats ≥25 nt, up to 50 nt) must be tested for reverse-complement concordance before accepting an IS candidate.

## When NOT to use

- The insertion sequences of interest lack ITRs — Palidis is 'currently only equipped with discovering ISs with ITRs', so reverse-complement ITR verification is inapplicable
- ITRs are shorter than the minimum ITR length threshold (e.g. 25 nt default; ITRs as short as 10 nt are missed at default settings)
- Sequences are long genomic contigs requiring general-purpose homology search rather than short paired-sequence concordance testing

## Inputs

- FASTA file of candidate ITR sequences from upstream Palidis matching step (getCandidateITRs)
- FASTA file of candidate IS region sequences
- CD-HIT-EST cluster assignments for candidate ITRs

## Outputs

- is_candidates.fasta containing IS candidates that pass cluster-concordance and reverse-complement verification
- blastn alignment results (plus/minus strand hits with identity counts) for paired ITRs

## How to apply

After clustering candidate ITR sequences with CD-HIT-EST (score mode G 0, identity threshold c 0.9, aL 0.0, aS 0.9), check each candidate IS to confirm its two flanking ITRs fall in the same cluster. Then align the paired ITRs with blastn using short-sequence settings: -task blastn -word_size 4, restricting the search to plus/minus strands. Retain the candidate only if the number of identities in the alignment is at least the minimum ITR length (default 25 nt in Palidis) and the total IS length is within the bounds min_is_len 500 to max_is_len 3000. Candidates passing both the cluster-concordance and reverse-complement checks are written to is_candidates.fasta.

## Related tools

- **blastn** (Aligns paired candidate ITRs on plus/minus strands with -task blastn -word_size 4 to verify they are reverse complements; requires identities ≥ minimum ITR length)
- **CD-HIT-EST** (Clusters candidate ITR sequences (c 0.9, aS 0.9, aL 0.0, G 0) upstream of the alignment check so that both ITRs of a candidate IS must fall in the same cluster)
- **Palidis** (Pipeline whose Step 3 (clusterReads + getITRs) orchestrates the clustering, length filtering, and blastn-based reverse-complement concordance check) — https://github.com/blue-moon22/palidis4

## Examples

```
makeblastdb -in itr_pairs.fasta -dbtype nucl; blastn -task blastn -word_size 4 -query itrA.fasta -subject itrB.fasta -strand both
```

## Evaluation signals

- Every retained IS candidate has both flanking ITRs assigned to the same CD-HIT-EST cluster (c 0.9, aS 0.9)
- For each accepted pair, the blastn (blastn task, word_size 4, plus/minus strands) identity count is ≥ the minimum ITR length (default 25)
- All output sequences in is_candidates.fasta have total length within 500–3000 bp (min_is_len/max_is_len)
- Running the Palidis regression test (./tests/regression_tests.sh) prints 'Test passed. All outputs expected.'

## Limitations

- ITRs as short as ten nucleotides exist biologically but are missed at the default minimum ITR length of 25 nt; the ISC catalogue is restricted to ITRs ≥25 nt
- ISs flanked by ITRs beyond the 500–3000 bp proximity range (e.g. regions lengthened by nested insertion events) are missed; increasing max_is_len may be needed in future iterations
- Running with a lower minimum ITR length threshold and smaller k-mer length is more computationally intensive, especially with complex mixed microbial genomes

## Evidence

- [other] Verify that paired ITRs are reverse complements by running blastn with -task blastn -word_size 4, restricting to plus/minus strands: [evidence span withheld — anchor status: not_found]
- [readme] Cluster candidate ITRs using CD-HIT-EST: "5. Cluster candidate ITRs using CD-HIT-EST [`clusterReads`] 6. Get putative ITRs by cluster concordance and output Insertion Sequences [`getITRs`]"
- [discussion] the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length: "the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis, although ITRs can be as short as ten nucleotides in length"
- [discussion] Palidis is currently only equipped with discovering ISs with ITRs.: "Palidis is currently only equipped with discovering ISs with ITRs."
- [other] identifies insertion sequences by ITRs that belong to the same cluster and are reverse complements of each other: [evidence span withheld — anchor status: not_found]
