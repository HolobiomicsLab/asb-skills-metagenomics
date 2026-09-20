---
name: cross-source-protein-sequence-integration
description: Use when constructing a reference protein database from coding genes
  of collected genomes plus curated entries (e.g. UniProt fungal subset), and redundancy
  and annotation quality must be controlled.
when_to_use_negative:
- When building a taxonomic marker-gene database — FunOMIC-T uses 99% identity CD-HIT
  clustering of single-copy marker genes instead of 95% protein clustering
- When input proteins are already non-redundant or pre-clustered; re-clustering with
  CD-HIT would discard legitimate near-identical sequences
- When functional annotation of metagenomic reads is not the goal, since ~1.5 million
  unannotated proteins are deliberately discarded here
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_3174
tools:
- name: CD-HIT
  role: Performs the 95% identity clustering that removes redundancy from the combined
    annotated protein set
- name: FunOMIC
  role: Pipeline whose built-in functional database (FunOMIC-P) is the output of this
    integration and is queried with Diamond blastx
  repo: https://github.com/ManichanhLab/FunOMIC
- name: Diamond blastx v2.0.8
  role: 'Downstream consumer: maps metagenomic reads against the FunOMIC-P database
    with e-value < 10e-10 for functional annotation'
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:0ca75acc06580ab51c9f6535c8d8a46f72faa5d724769fac919b00ab4c6d72cf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  repo_url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - CD-HIT
  - Diamond blastx v2.0.8
  - FunOMIC
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0623
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cross-source-protein-sequence-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build a non-redundant functional protein reference (e.g. FunOMIC-P) by integrating amino-acid sequences from multiple repositories (NCBI, JGI, UniProt), filtering to explicitly annotated proteins, and clustering at 95% identity with CD-HIT.

## When to use

Use when constructing a reference protein database from coding genes of collected genomes plus curated entries (e.g. UniProt fungal subset), and redundancy and annotation quality must be controlled.

## When NOT to use

- When building a taxonomic marker-gene database — FunOMIC-T uses 99% identity CD-HIT clustering of single-copy marker genes instead of 95% protein clustering
- When input proteins are already non-redundant or pre-clustered; re-clustering with CD-HIT would discard legitimate near-identical sequences
- When functional annotation of metagenomic reads is not the goal, since ~1.5 million unannotated proteins are deliberately discarded here

## Inputs

- Amino-acid coding sequences from quality-filtered fungal genomes downloaded from NCBI and JGI MycoCosm
- Reviewed fungal protein sequences from the UniProt fungal subset (35,360 entries)
- Functional annotations (EC numbers, KEGG pathways) for the source proteins

## Outputs

- Non-redundant protein database in FASTA format (FunOMIC-P: 3,413,239 sequences)
- Cluster summary table reporting the reduction from ~4.9 million input genes
- Accession-to-annotation mapping (EC numbers, KEGG pathways) for the retained proteins

## How to apply

Collect amino-acid sequences corresponding to the quality-filtered genomes (2,967 of 4,331 in FunOMIC) plus reviewed proteins from UniProt (35,360 fungal entries), yielding a combined set of ~4.9 million genes. Discard all proteins lacking an explicit functional annotation (~1.5 million removed) so the database supports downstream functional profiling. Run CD-HIT with a 95% identity clustering threshold to remove redundancy, producing the final non-redundant set of 3,413,239 fungal protein sequences (FunOMIC-P) in FASTA. Link protein accessions from each source (JGI, NCBI, UniProt) to EC numbers and KEGG pathways so metagenomic hits translate into functional annotations.

## Related tools

- **CD-HIT** (Performs the 95% identity clustering that removes redundancy from the combined annotated protein set)
- **FunOMIC** (Pipeline whose built-in functional database (FunOMIC-P) is the output of this integration and is queried with Diamond blastx) — https://github.com/ManichanhLab/FunOMIC
- **Diamond blastx v2.0.8** (Downstream consumer: maps metagenomic reads against the FunOMIC-P database with e-value < 10e-10 for functional annotation) — https://github.com/bbuchfink/diamond

## Examples

```
cd-hit -i combined_annotated_proteins.fasta -o FunOMIC-P.fasta -c 0.95 -d 0
```

## Evaluation signals

- Output FASTA contains exactly 3,413,239 non-redundant fungal protein sequences
- Cluster summary table reports the expected reduction from ~4.9 million input genes after the 1.5 million unannotated proteins were discarded and 95% identity clustering applied
- All retained sequences carry an explicit functional annotation (EC number or KEGG pathway linkage)
- Source accession provenance (JGI, NCBI, UniProt) is preserved in headers or a side mapping for downstream annotation lookup

## Limitations

- High inter-kingdom conservation of protein-coding genes means bacterial contamination was not totally preventable even with strict mapping thresholds (coverage >95%, identity >99%, e-value < 10e-10)
- No bacterial read filtration step is included before functional annotation; the authors propose adding it in a future update
- Database coverage is bounded by the completeness of available fungal genomes and the requirement for explicit annotation, excluding ~1.5 million uncharacterized proteins

## Evidence

- [methods] Functional database construction (95% identity clustering): [evidence span withheld — anchor status: not_found]
