---
name: tsv-summary-reporting
description: Use when running Plassembler hybrid plasmid assembly and you need a machine-readable
  per-contig table capturing circularity and PLSDB mash-screening matches for downstream
  validation or reporting.
when_to_use_negative:
- The Plassembler PLSDB database (install_database.py) has not been installed, so
  no mash sketch database exists to screen contigs against
- The contigs come from an assembly method that does not supply Unicycler-style circularity
  information, making the circularity columns unreliable
- The goal is read filtering or assembly rather than summarizing already-assembled
  plasmid contigs
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Mash
  role: Sketches plasmid contigs and computes MinHash distances against the PLSDB
    sketch database to find the top hit under the 0.1 distance threshold
  repo: https://github.com/marbl/Mash
- name: PLSDB
  role: Reference plasmid database screened via mash to annotate each contig with
    its closest known plasmid match
- name: Plassembler
  role: Implements the threshold filtering (similarity 0.9 / distance 0.1), circularity
    determination, and writes plasmid_summary.tsv
  repo: https://github.com/gbouras13/plassembler
- name: Python
  role: Language in which Plassembler's summary-table logic and database installation
    are implemented
  repo: https://github.com/gbouras13/plassembler
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:9e287b0398ae4ac65a3a7aa24f2da37b44bee0443011aec3f11d8a932a4d2a11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marbl/Mash
  tools:
  - Mash
  - PLSDB
  - Plassembler
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# tsv-summary-reporting

## Summary

Aggregate Plassembler's per-contig plasmid results into a single tab-separated summary table (plasmid_summary.tsv) that pairs each assembled plasmid contig's Unicycler-derived circularity status with its top PLSDB mash hit below the 0.1 distance threshold.

## When to use

Use when running Plassembler hybrid plasmid assembly and you need a machine-readable per-contig table capturing circularity and PLSDB mash-screening matches for downstream validation or reporting.

## When NOT to use

- The Plassembler PLSDB database (install_database.py) has not been installed, so no mash sketch database exists to screen contigs against
- The contigs come from an assembly method that does not supply Unicycler-style circularity information, making the circularity columns unreliable
- The goal is read filtering or assembly rather than summarizing already-assembled plasmid contigs

## Inputs

- Unicycler-assembled plasmid contig FASTA files from the Plassembler output directory
- PLSDB sketch (mash .msh) database from the Plassembler database installed by install_database.py
- Per-contig circularity flags determined from the Unicycler assembly
- Per-contig mash dist results against PLSDB

## Outputs

- plasmid_summary.tsv: a tab-separated file with one row per assembled plasmid contig, including circularity status and PLSDB top-hit match columns

## How to apply

After Unicycler assembles the extracted plasmid reads, sketch each assembled plasmid contig and run mash dist against the PLSDB sketch database shipped with the Plassembler database (installed via install_database.py). For each contig, filter hits using Plassembler's thresholds (similarity 0.9 / mash distance 0.1) and retain the lowest-distance hit as the PLSDB top match; contigs with no hit under the threshold are reported as unmatched. Determine circularity per contig from the Unicycler assembly graph and write one tab-separated row per contig to plasmid_summary.tsv, combining the circularity and PLSDB match columns. The rationale is that mash gives a fast MinHash similarity screen against PLSDB, and the summary table is the single artifact that records both assembly topology (circular vs linear) and database match evidence for every output contig.

## Related tools

- **Mash** (Sketches plasmid contigs and computes MinHash distances against the PLSDB sketch database to find the top hit under the 0.1 distance threshold) — https://github.com/marbl/Mash
- **PLSDB** (Reference plasmid database screened via mash to annotate each contig with its closest known plasmid match)
- **Plassembler** (Implements the threshold filtering (similarity 0.9 / distance 0.1), circularity determination, and writes plasmid_summary.tsv) — https://github.com/gbouras13/plassembler
- **Python** (Language in which Plassembler's summary-table logic and database installation are implemented) — https://github.com/gbouras13/plassembler

## Examples

```
mash dist plsdb.msh assembled_plasmids.fasta > plsdb_hits.tsv
```

## Evaluation signals

- plasmid_summary.tsv exists and is a valid tab-separated file
- The table contains exactly one row per assembled plasmid contig, with contig IDs matching the input FASTA
- Circularity and PLSDB match columns are populated for every contig row
- Every reported PLSDB match satisfies the mash distance threshold of 0.1 (similarity 0.9)
- Contigs with no PLSDB hit below 0.1 distance are represented as unmatched rather than dropped from the table

## Limitations

- PLSDB top hits are MinHash (mash) similarity matches, not curated functional annotations; a hit only indicates sequence similarity below the 0.1 distance threshold
- Plasmid database annotation depends on the Plassembler database being installed via install_database.py; without it, PLSDB match columns cannot be populated
- Linear plasmids are a known weak point of the Unicycler-based approach, so circularity status in the summary may be unreliable for linear plasmids

## Evidence

- [methods] mash screening of contigs against PLSDB: "`plassembler` uses [mash](https://github.com/marbl/Mash) as a quick way to determine whether each assembled contig has any similar hits in [PLSDB]"
- [methods] plasmid copy number calculation: "plassembler calculates plasmid copy number for both long and short read sets."
- [discussion] PLSDB database mapping functionality: "Adds install_database.py, the Plassembler database and functionality for mapping plasmid contigs to PLDSB."
