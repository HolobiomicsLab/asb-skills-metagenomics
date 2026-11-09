---
name: fasta-sequence-filtering
description: Use when a large FASTA of predicted CDSs (e.g. ~5.5 million from assembled
  vaginal metagenomes) must be collapsed to a non-redundant gene set before annotation
  or read mapping.
when_to_use_negative:
- Input is already a non-redundant gene catalog (e.g. the published VIRGO 0.95-million-gene
  database) rather than raw predicted CDSs.
- Protein-level clustering is the goal; this nucleotide-level procedure (CD-HIT-EST)
  is for CDS de-redundancy, while VIRGO's VOGs are built separately by Jaccard clustering
  of all-vs-all BLASTP results.
- Reads rather than assembled/predicted sequences need filtering; read-level cutoffs
  in VIRGO (e.g. minimum 2 reads per gene hit, bowtie -l 25 seed length) are a different
  filter layer.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: CD-HIT-EST
  role: Greedy incremental nucleotide sequence clustering used to de-replicate CDSs
    at 95% identity over 90% of the shorter gene
- name: VIRGO
  role: Gene catalog pipeline whose predicted CDSs are de-redundified and whose downstream
    annotation and mapping modules consume the resulting V-prefixed gene IDs
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:58195b6add897e0ec83b09a5d73e81253245b9a8826a22f73d3101b79742b8b7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-EST
  - VIRGO
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# fasta-sequence-filtering

## Summary

De-redundify predicted coding sequences from assembled vaginal metagenomes with CD-HIT-EST at 95% identity over 90% of the shorter sequence, retaining only CDSs longer than 99 bp, to build a non-redundant gene catalog such as VIRGO's 948,158-gene catalog.

## When to use

Use when a large FASTA of predicted CDSs (e.g. ~5.5 million from assembled vaginal metagenomes) must be collapsed to a non-redundant gene set before annotation or read mapping.

## When NOT to use

- Input is already a non-redundant gene catalog (e.g. the published VIRGO 0.95-million-gene database) rather than raw predicted CDSs.
- Protein-level clustering is the goal; this nucleotide-level procedure (CD-HIT-EST) is for CDS de-redundancy, while VIRGO's VOGs are built separately by Jaccard clustering of all-vs-all BLASTP results.
- Reads rather than assembled/predicted sequences need filtering; read-level cutoffs in VIRGO (e.g. minimum 2 reads per gene hit, bowtie -l 25 seed length) are a different filter layer.

## Inputs

- Nucleotide FASTA of predicted CDSs (~5.5 million coding sequences from assembled vaginal metagenomes)

## Outputs

- Non-redundant gene catalog FASTA (nr_gene_catalog.fasta) with 948,158 representative CDSs longer than 99 bp, retaining V-prefixed gene IDs

## How to apply

Load the predicted CDS nucleotide FASTA and first discard sequences shorter than 99 bp to avoid spurious clusters of very short fragments. Run CD-HIT-EST v4.6 in greedy incremental mode with sequence identity threshold -c 0.95 and alignment coverage of the shorter sequence -aS 0.90, so clusters require >=95% identity over >=90% of the shorter gene length. Write the cluster representative (longest) sequences to the non-redundant gene catalog FASTA, preserving V-prefixed gene IDs used by downstream VIRGO annotation and mapping modules. Validate that the output contains 948,158 sequences, about 17.2% of the original 5.5 million predicted CDSs, matching the reported 0.95-million-gene VIRGO catalog.

## Related tools

- **CD-HIT-EST** (Greedy incremental nucleotide sequence clustering used to de-replicate CDSs at 95% identity over 90% of the shorter gene)
- **VIRGO** (Gene catalog pipeline whose predicted CDSs are de-redundified and whose downstream annotation and mapping modules consume the resulting V-prefixed gene IDs) — github.com/ravel-lab/VIRGO

## Examples

```
cd-hit-est -i predicted_cds.fasta -o nr_gene_catalog.fasta -c 0.95 -aS 0.90 -g 1 -d 0
```

## Evaluation signals

- Output FASTA contains exactly 948,158 sequences for the full VIRGO build.
- Retained genes represent ~17.2% of the original 5.5 million predicted CDSs.
- All output sequences are >= 99 bp.
- Sequence headers preserve V-prefixed VIRGO gene IDs so downstream modules (mapping, annotation grep) resolve them.
- Every cluster representative is the longest sequence of its cluster, consistent with greedy incremental CD-HIT-EST behavior.

## Limitations

- Greedy incremental clustering is order-dependent; representatives depend on input sequence order rather than global optimal clustering.
- Very short CDS fragments (<99 bp) are discarded, so genuine short genes could be excluded.
- The catalog is primarily focused on bacteria, with limited inclusion of viral and fungal gene sequences (only 10,908 fungal and 15,965 bacteriophage genes carried separately).

## Evidence

- [methods] Discard CDS sequences shorter than 99 bp to avoid spurious clusters of very short fragments.: [evidence span withheld — anchor status: not_found]
- [methods] clusters require >=95% identity over >=90% of the shorter gene length: [evidence span withheld — anchor status: not_found]
- [results] 948,158 non-redundant CDSs: [evidence span withheld — anchor status: not_found]
- [abstract] assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO): "we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes"
- [discussion] The database is primarily focused on bacteria: "The database is primarily focused on bacteria with limited inclusion of viral and fungal gene sequences."
