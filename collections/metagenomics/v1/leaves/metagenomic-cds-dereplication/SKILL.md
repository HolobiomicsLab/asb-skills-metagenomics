---
name: metagenomic-cds-dereplication
description: Use when building a body-site non-redundant gene catalog from a large
  set of predicted CDSs (nucleotide FASTA) and redundant near-identical sequences
  across metagenomes must be collapsed into cluster representatives for downstream
  annotation and read mapping.
when_to_use_negative:
- Input sequences are already de-replicated or you need orthologous protein families
  rather than nucleotide-level non-redundancy — use the VIRGO VOG Jaccard clustering
  pipeline instead
- Pairs of sequences share >=95% identity but over <90% of the shorter gene and should
  be retained as distinct (e.g. partial vs. extended gene models) — the -aS 0.90 coverage
  rule would wrongly collapse/cluster them differently
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: CD-HIT-EST
  role: Greedy incremental nucleotide clustering tool that performs the de-redundancy
    at 95% identity over 90% of the shorter gene length
- name: VIRGO
  role: Pipeline whose nr_gene_catalog.fasta output this de-redundancy step produces;
    V-prefixed representative IDs feed VIRGO's annotation and mapping modules
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:0058860e2a67664ed4ebd500a81b0933f0c1116fa51252894ac313bda604f0eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-EST
  - VIRGO
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# metagenomic-cds-dereplication

## Summary

Greedy nucleotide-level de-redundancy of predicted coding sequences with CD-HIT-EST to build a non-redundant gene catalog (VIRGO), collapsing ~5.5 million predicted CDSs from assembled vaginal metagenomes into 948,158 non-redundant representatives.

## When to use

Use when building a body-site non-redundant gene catalog from a large set of predicted CDSs (nucleotide FASTA) and redundant near-identical sequences across metagenomes must be collapsed into cluster representatives for downstream annotation and read mapping.

## When NOT to use

- Input sequences are already de-replicated or you need orthologous protein families rather than nucleotide-level non-redundancy — use the VIRGO VOG Jaccard clustering pipeline instead
- Pairs of sequences share >=95% identity but over <90% of the shorter gene and should be retained as distinct (e.g. partial vs. extended gene models) — the -aS 0.90 coverage rule would wrongly collapse/cluster them differently

## Inputs

- Predicted CDS nucleotide FASTA (~5.5 million coding sequences from assembled vaginal metagenomes)
- Minimum gene-length cutoff parameter (99 bp)

## Outputs

- nr_gene_catalog.fasta containing 948,158 non-redundant CDS representatives (V-prefixed gene IDs, all >99 bp)
- Non-redundant gene catalog (0.95 million genes) forming the VIRGO database for downstream annotation and read mapping

## How to apply

Load the predicted CDS nucleotide FASTA (~5.5 million coding sequences predicted from assembled vaginal metagenomes) and first discard sequences shorter than 99 bp to avoid spurious clusters of very short fragments. Run CD-HIT-EST v4.6 in greedy incremental mode with sequence identity threshold -c 0.95 and -aS 0.90, so each cluster requires >=95% identity over >=90% of the shorter gene length. Write the cluster representative (longest) sequences to nr_gene_catalog.fasta as the non-redundant gene catalog, preserving V-prefixed gene IDs used by downstream VIRGO annotation and mapping modules. The greedy incremental clustering orders sequences by decreasing length, so the longest member of each cluster becomes the representative. The resulting catalog of 948,158 non-redundant genes (~17.2% of input) is the VIRGO reference that is then taxonomically and functionally annotated and made bowtie/BLAST-searchable.

## Related tools

- **CD-HIT-EST** (Greedy incremental nucleotide clustering tool that performs the de-redundancy at 95% identity over 90% of the shorter gene length)
- **VIRGO** (Pipeline whose nr_gene_catalog.fasta output this de-redundancy step produces; V-prefixed representative IDs feed VIRGO's annotation and mapping modules) — github.com/ravel-lab/VIRGO

## Examples

```
cd-hit-est -i all_predicted_cds.fasta -o nr_gene_catalog.fasta -c 0.95 -aS 0.90 -g 1 -d 0
```

## Evaluation signals

- Output FASTA contains 948,158 sequences, ~17.2% of the original ~5.5 million predicted CDSs
- All retained sequences are longer than 99 bp
- Every cluster representative is the longest (or equal-length) member of its CD-HIT-EST cluster
- Gene IDs in nr_gene_catalog.fasta retain the V-prefix format used by VIRGO downstream modules (e.g. V1593031)
- No two sequences in the output share >=95% identity over >=90% of the shorter sequence (spot-check with all-vs-all search)

## Limitations

- The gene-level de-redundancy discards redundant copies, so within-species/intra-community strain variants at >=95% identity over >=90% coverage are collapsed into a single representative; VIRGO's reference-based approach has acknowledged limitations
- Very short CDSs (<99 bp) are discarded before clustering, so sub-99-bp genes are absent from the catalog
- CD-HIT greedy incremental clustering is order- and length-dependent; representatives are the longest sequences, not necessarily the most biologically representative

## Evidence

- [methods] The greedy pairwise nucleotide-level de-redundancy procedure (95% identity over 90% of the shorter gene length) yielded a catalog of 948,158 non-redundant CDSs longer than 99 bp: [evidence span withheld — anchor status: not_found]
- [methods] Run CD-HIT-EST v4.6 in greedy incremental mode with sequence identity threshold -c 0.95 and alignment coverage of the shorter sequence -aS 0.90: [evidence span withheld — anchor status: not_found]
- [abstract] we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes: "we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes"
- [readme] VIRGO is a non-redundant gene catalog for the microbial communities that inhabit the human vagina. It was built using a combination of metagenomic and urogential isolate genome sequences.: "VIRGO is a non-redundant gene catalog for the microbial communities that inhabit the human vagina. It was built using a combination of metagenomic"
