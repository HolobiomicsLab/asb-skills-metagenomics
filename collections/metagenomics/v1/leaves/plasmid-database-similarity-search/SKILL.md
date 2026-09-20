---
name: plasmid-database-similarity-search
description: Use when assembled plasmid contigs (e.g. Unicycler output FASTA) need
  to be matched against a known-plasmid reference database such as PLSDB to annotate
  novelty or similarity.
when_to_use_negative:
- When contigs need full gene-level functional annotation rather than database-level
  similarity (Mash gives MinHash distance, not gene content)
- When plasmid contigs have not yet been assembled/extracted (this skill operates
  on assembled FASTA contigs, not raw reads)
- When a chromosome-contig classification is needed — mash PLSDB matching is for plasmid
  similarity, not chromosomal read removal
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Mash
  role: Sketches plasmid contigs and computes MinHash distances (mash dist) against
    the PLSDB sketch database
  repo: https://github.com/marbl/Mash
- name: Plassembler
  role: Orchestrates the PLSDB top-hit selection (distance threshold 0.1 / similarity
    0.9) and writes plasmid_summary.tsv; ships the database via install_database.py
  repo: https://github.com/gbouras13/plassembler
- name: PLSDB
  role: Reference plasmid database of known plasmid sequences screened via mash
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:270316139b9ff6f35035996044c8f52b05f0e15f59f2ba5ba6181ce88db896db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marbl/Mash
  tools:
  - Mash
  - PLSDB
  - Plassembler
  - Python
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# plasmid-database-similarity-search

## Summary

Screens assembled plasmid contigs against the PLSDB database using Mash MinHash distances, keeping the lowest-distance PLSDB hit per contig as the top match. Plassembler uses this to annotate whether each assembled plasmid resembles known plasmids.

## When to use

Use when assembled plasmid contigs (e.g. Unicycler output FASTA) need to be matched against a known-plasmid reference database such as PLSDB to annotate novelty or similarity.

## When NOT to use

- When contigs need full gene-level functional annotation rather than database-level similarity (Mash gives MinHash distance, not gene content)
- When plasmid contigs have not yet been assembled/extracted (this skill operates on assembled FASTA contigs, not raw reads)
- When a chromosome-contig classification is needed — mash PLSDB matching is for plasmid similarity, not chromosomal read removal

## Inputs

- Assembled plasmid contig FASTA files (Unicycler output)
- PLSDB Mash sketch database from the Plassembler database package (install_database.py)
- Unicycler assembly information for circularity status

## Outputs

- Per-contig PLSDB top-hit assignments (lowest mash distance per contig)
- plasmid_summary.tsv with per-contig circularity and PLSDB match columns

## How to apply

Load the assembled plasmid contigs (Unicycler output FASTA) and the PLSDB sketch database shipped with the Plassembler database (installed via install_database.py). Sketch each plasmid contig and run mash dist against the PLSDB sketch database to compute pairwise MinHash distances for every contig-database pair. For each contig, filter hits with the Plassembler distance thresholds (similarity 0.9 / distance 0.1) and keep the lowest-distance hit as the top PLSDB match. Annotate each contig with its circularity status as determined from the Unicycler assembly, and write all per-contig circularity and PLSDB match columns to a summary table plasmid_summary.tsv.

## Related tools

- **Mash** (Sketches plasmid contigs and computes MinHash distances (mash dist) against the PLSDB sketch database) — https://github.com/marbl/Mash
- **Plassembler** (Orchestrates the PLSDB top-hit selection (distance threshold 0.1 / similarity 0.9) and writes plasmid_summary.tsv; ships the database via install_database.py) — https://github.com/gbouras13/plassembler
- **PLSDB** (Reference plasmid database of known plasmid sequences screened via mash)

## Examples

```
mash dist assembled_plasmids.fasta PLSDB.msh
```

## Evaluation signals

- plasmid_summary.tsv exists and is tab-separated
- plasmid_summary.tsv contains one row per assembled plasmid contig
- Circularity and PLSDB match columns are populated for each contig
- Every reported PLSDB match has a mash distance below the 0.1 threshold (equivalently similarity >= 0.9)
- Each contig's reported match is the minimum-distance hit among all PLSDB entries passing the threshold

## Limitations

- Contigs with no PLSDB entry below the 0.1 mash distance threshold receive no match, so truly novel plasmids cannot be annotated against PLSDB
- The Plassembler database/PLSDB mapping was only introduced in a later release (v1.3.0 adds install_database.py), so older versions lack this functionality

## Evidence

- [methods] Mash screening of contigs against PLSDB: "`plassembler` uses [mash](https://github.com/marbl/Mash) as a quick way to determine whether each assembled contig has any similar hits in [PLSDB]"
- [discussion] Plassembler database installation and PLSDB mapping: "Adds install_database.py, the Plassembler database and functionality for mapping plasmid contigs to PLDSB."
- [methods] Hybrid assembly feeding the annotation pipeline: "These reads are then assembled using Unicycler (Wick et al 2017)."
