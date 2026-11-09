---
name: protein-accession-pathway-linking
description: Use when constructing a functional protein reference database where protein
  sequences must carry EC-number and pathway metadata (e.g., KEGG) so that downstream
  read-mapping yields interpretable functional profiles.
when_to_use_negative:
- When proteins lack explicit functional annotations and no pathway resources (e.g.,
  KEGG via KEGGREST) are available — the article's approach discards ~1.5 million
  unannotated proteins rather than predicting functions
- When building a taxonomic (marker-gene) database such as FunOMIC-T, which uses 99%
  identity CD-HIT clustering of single-copy marker genes instead of protein functional
  annotation
- When the input is already a non-redundant, functionally annotated protein database
  — re-clustering would be redundant
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: FunOMIC
  role: Pipeline that integrates genome coding genes and UniProt fungal proteins into
    FunOMIC-P and links accessions to EC numbers and KEGG pathways
  repo: https://github.com/ManichanhLab/FunOMIC
- name: CD-HIT
  role: 95% identity clustering to remove redundancy from the annotated protein set
    before accession-pathway linking is finalized
- name: KEGGREST
  role: R package required by the FunOMIC pipeline for handling KEGG pathway identifiers
    during database construction
  repo: https://www.bioconductor.org/packages/release/bioc/html/KEGGREST.html
- name: Diamond blastx v2.0.8
  role: Maps metagenomic reads to the FunOMIC-P database (coverage >95%, identity
    >99%, e-value < 10e-10) and resolves hits via the accession-to-pathway links
  repo: https://github.com/bbuchfink/diamond/wiki
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:e45207353ac3e34722ca5b80209d4670bc22f55a6acbac50d5f7793ff6e9f09f
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
  - KEGGREST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# protein-accession-pathway-linking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A step in building the FunOMIC-P functional mycobiome database: after clustering annotated fungal proteins, each accession (from JGI, NCBI, or UniProt) is linked to explicit functional metadata such as EC numbers and KEGG pathways so metagenomic reads can be functionally annotated by DIAMOND blastx.

## When to use

Use when constructing a functional protein reference database where protein sequences must carry EC-number and pathway metadata (e.g., KEGG) so that downstream read-mapping yields interpretable functional profiles.

## When NOT to use

- When proteins lack explicit functional annotations and no pathway resources (e.g., KEGG via KEGGREST) are available — the article's approach discards ~1.5 million unannotated proteins rather than predicting functions
- When building a taxonomic (marker-gene) database such as FunOMIC-T, which uses 99% identity CD-HIT clustering of single-copy marker genes instead of protein functional annotation
- When the input is already a non-redundant, functionally annotated protein database — re-clustering would be redundant

## Inputs

- Amino-acid (coding-gene) sequences from quality-filtered fungal genomes (NCBI, JGI MycoCosm) in FASTA format
- Reviewed fungal protein sequences from the UniProt fungal subset
- Pre-existing functional annotations for those proteins (EC numbers, KEGG pathway identifiers)

## Outputs

- FunOMIC-P database: 3,413,239 non-redundant fungal protein sequences in FASTA format
- Accession-to-function mapping linking protein accessions (JGI/NCBI/UniProt) to EC numbers and KEGG pathways

## How to apply

Collect amino-acid sequences for fungal genomes (e.g., 2,967 quality-filtered genomes from NCBI and JGI MycoCosm) plus reviewed fungal proteins from UniProt, forming a combined set (~4.9 million genes). Discard all proteins lacking an explicit functional annotation (about 1.5 million removed), since the linking step depends on existing EC/KEGG annotations rather than de novo prediction. Remove redundancy with CD-HIT at a 95% identity threshold, yielding 3,413,239 non-redundant fungal proteins (FunOMIC-P) in FASTA format, then propagate each protein's source accession to EC numbers and KEGG pathways; the pipeline uses the R KEGGREST package to handle KEGG identifiers. Downstream, DIAMOND blastx maps reads against this FASTA (coverage >95%, identity >99%, e-value < 10e-10) and resolves hits back to the accession-to-pathway links for functional profiling.

## Related tools

- **FunOMIC** (Pipeline that integrates genome coding genes and UniProt fungal proteins into FunOMIC-P and links accessions to EC numbers and KEGG pathways) — https://github.com/ManichanhLab/FunOMIC
- **CD-HIT** (95% identity clustering to remove redundancy from the annotated protein set before accession-pathway linking is finalized)
- **KEGGREST** (R package required by the FunOMIC pipeline for handling KEGG pathway identifiers during database construction) — https://www.bioconductor.org/packages/release/bioc/html/KEGGREST.html
- **Diamond blastx v2.0.8** (Maps metagenomic reads to the FunOMIC-P database (coverage >95%, identity >99%, e-value < 10e-10) and resolves hits via the accession-to-pathway links) — https://github.com/bbuchfink/diamond/wiki

## Examples

```
FunOMIC.sh -1 read1.fastq -2 read2.fastq -p output_prefix -o output_folder -a bacterialDB_path -b FunOMIC-T_folder -c FunOMIC-P_folder -t #_threads
```

## Evaluation signals

- Verify the output FASTA contains exactly 3,413,239 non-redundant fungal protein sequences
- Check the cluster summary table reports the expected reduction from ~4.9 million input genes after 95% identity CD-HIT clustering
- Confirm ~1.5 million proteins lacking explicit annotation were discarded before clustering
- Verify each retained protein accession resolves to at least one EC number and/or KEGG pathway entry
- Test that DIAMOND blastx hits against FunOMIC-P can be traced back to source accessions (JGI, NCBI, or UniProt)

## Limitations

- High inter-kingdom conservation of protein-coding genes means bacterial contamination was not totally preventable, even with the strict DIAMOND mapping threshold (coverage >95%, identity >99%, e-value < 10e-10)
- No bacterial read filtration step is included before functional annotation; removal was only proposed for a future update of the tool
- Proteins without explicit functional annotation (about 1.5 million) are discarded, so proteins with unknown function cannot contribute pathway links

## Evidence

- [methods] Accessions linked to EC/KEGG: [evidence span withheld — anchor status: not_found]
- [methods] 95% clustering yields FunOMIC-P: [evidence span withheld — anchor status: not_found]
