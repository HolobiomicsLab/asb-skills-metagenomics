---
name: sequence-coverage-metrics-computation
description: Use when benchmarking alignment-based marker retrieval (e.g., Patchwork/DIAMOND)
  against a known set of reference USCOs and you need percent identity, query coverage,
  and a combined pidentical,cov score per marker.
when_to_use_negative:
- References are not near-USCOs — the article recommends limiting reference sequences
  to near-USCOs to avoid chimeric sequences, which would invalidate coverage/identity
  comparisons
- Query and reference are the same species with expected near-perfect recovery — use
  the same-species benchmark (815/815 retrieved, 95.9% identity) rather than the divergent-reference
  metric
- You lack a defined set of homologous reference USCOs to restrict evaluation to,
  since the metric is only meaningful against known homologs
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3293
tools:
- name: Patchwork
  role: Retrieves and concatenates phylogenetic markers from the query assembly/read
    data against amino acid reference sequences, producing the recovered markers evaluated
    by the metrics
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Performs the BLASTX similarity search (BLOSUM62, gap open 11, gap extension
    1) used both for marker retrieval and for the --ultra-sensitive back-alignment
    that defines recoverable USCOs and per-marker identity/coverage
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Implementation language of Patchwork (v1.8.2) in which the metric computation
    pipeline runs
  repo: https://github.com/fethalen/Patchwork
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:85febb2731cdd8ff76280533f8b26aa3694fa2b3bfc34ce5d16f042f82df6c3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# sequence-coverage-metrics-computation

## Summary

Compute the combined retrieval-quality metric pidentical,cov = nmatch/naligned x cov used by the Patchwork benchmark, where cov is total recovered marker length divided by total reference USCO length, to quantify marker retrieval accuracy across divergent species pairs.

## When to use

Use when benchmarking alignment-based marker retrieval (e.g., Patchwork/DIAMOND) against a known set of reference USCOs and you need percent identity, query coverage, and a combined pidentical,cov score per marker.

## When NOT to use

- References are not near-USCOs — the article recommends limiting reference sequences to near-USCOs to avoid chimeric sequences, which would invalidate coverage/identity comparisons
- Query and reference are the same species with expected near-perfect recovery — use the same-species benchmark (815/815 retrieved, 95.9% identity) rather than the divergent-reference metric
- You lack a defined set of homologous reference USCOs to restrict evaluation to, since the metric is only meaningful against known homologs

## Inputs

- Retrieved marker FASTA sequences from Patchwork (e.g., against 957 H. robusta near-USCOs)
- Reference USCO protein sequences (GenBank GCA_000326865.1 for H. robusta; GCA_904063045.1 for D. gyrociliatus 815 USCOs)
- DIAMOND --ultra-sensitive search results (marker-to-reference alignments with bit scores)
- Set of 'recoverable' query USCOs (778 D. gyrociliatus USCOs with a reference hit)

## Outputs

- Per-marker metrics table (patchwork_h_robusta_benchmark_metrics.tsv) with percent identical positions, reference coverage, and combined pidentical,cov
- Retrieval counts (e.g., 943 of 957 references retrieved; 769 of 778 uniquely aligned markers)
- Aggregate means: mean percent identity (~89%), mean query coverage (~74.5%), combined metric (~67.2%)

## How to apply

After running Patchwork (e.g., v0.5.1 with DIAMOND v2.0.13, --ultra-sensitive --frameshift 15 --masking 0) to retrieve markers against a reference set such as the 957 H. robusta near-USCOs (GenBank GCA_000326865.1), restrict evaluation to 'recoverable' query USCOs — for D. gyrociliatus (GCA_904063045.1) this means the 778 USCOs producing a hit under DIAMOND --ultra-sensitive. Search recovered markers against this restricted reference set with DIAMOND --ultra-sensitive, retaining only the highest bit-score marker per reference sequence. For each reference, compute nmatch/naligned (percent identical aligned positions) and cov (total recovered marker length / total reference USCO length), then combine as pidentical,cov = nmatch/naligned x cov. Aggregate per-marker values into a TSV metrics table and compare against published benchmark values (943/957 retrieved, 769 of 778 uniquely aligned, ~89% mean identity, ~74.5% mean coverage, ~67.2% combined metric).

## Related tools

- **Patchwork** (Retrieves and concatenates phylogenetic markers from the query assembly/read data against amino acid reference sequences, producing the recovered markers evaluated by the metrics) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Performs the BLASTX similarity search (BLOSUM62, gap open 11, gap extension 1) used both for marker retrieval and for the --ultra-sensitive back-alignment that defines recoverable USCOs and per-marker identity/coverage) — https://github.com/bbuchfink/diamond
- **Julia** (Implementation language of Patchwork (v1.8.2) in which the metric computation pipeline runs) — https://github.com/fethalen/Patchwork

## Examples

```
diamond blastx -d h_robusta_near_usco.dmnd -q d_gyrociliatus_spades_k55.fasta --ultra-sensitive --frameshift 15 --masking 0 -o patchwork_hits.tsv
```

## Evaluation signals

- Retrieval count must match the published benchmark: 943 of 957 H. robusta reference sequences retrieved
- Exactly 769 of the 778 recoverable D. gyrociliatus USCOs align back to 1 and only 1 reference
- Mean percent identical aligned positions is approximately 89%
- Mean query coverage is approximately 74.5%
- Combined pidentical,cov over all reference positions is approximately 67.2%

## Limitations

- Divergent references reduce recovery: only 943/957 references and 769/778 unique back-alignments were achieved even with --ultra-sensitive
- Evaluation must be restricted to 'recoverable' USCOs (those with a DIAMOND --ultra-sensitive hit); including non-recoverable markers would bias coverage downward
- Coverage estimates depend on disabling tantan masking (--masking 0), which the paper reports improves query coverage especially for read data
- Highest bit-score marker per reference must be retained; multi-mapping markers otherwise confound unique-alignment counts

## Evidence

- [results] Against distant H. robusta references, Patchwork retrieved 943 of 957 sequences; 769 aligned back uniquely to homologous D. gyrociliatus USCOs, with 89% average percent identity and 74.5% mean query coverage (67.2% over all reference positions): "We retrieved 943 out of the 957 H. robusta reference sequences. Of these, 769 successfully aligned back to 1 and only 1 of the 778 D. gyrociliatus USCOs"
- [results] The metric formula combines identity and coverage over reference positions: [evidence span withheld — anchor status: not_found]
- [methods] Benchmark parameters for the divergent-reference search: [evidence span withheld — anchor status: not_found]
- [results] Masking improves query coverage, a prerequisite for good metric values: "query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument)"
- [readme] README description of Patchwork's core operation underlying the metric inputs: "Patchwork is an alignment-based program for retrieving and concatenating phylogenetic markers from whole-genome sequencing (WGS) data."
