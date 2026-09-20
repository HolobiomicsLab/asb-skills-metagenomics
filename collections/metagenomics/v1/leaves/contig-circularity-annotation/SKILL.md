---
name: contig-circularity-annotation
description: Use when assembled plasmid contigs (e.g. Unicycler FASTA output from
  Plassembler) need per-contig circularity and known-plasmid similarity annotation
  against PLSDB, summarized into a plasmid_summary.tsv table.
when_to_use_negative:
- The contigs are not plasmid contigs from a Plassembler/Unicycler hybrid assembly
  and no circularity information from the Unicycler graph is available
- A full functional/gene annotation of plasmid contigs is needed rather than a nearest-known-plasmid
  (PLSDB) similarity call
- The analysis does not use the Plassembler database, so no PLSDB sketch database
  (from install_database.py) is available for mash screening
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Plassembler
  role: Implements the mash screening against PLSDB, threshold filtering, circularity
    extraction from the Unicycler assembly, and writes plasmid_summary.tsv
  repo: https://github.com/gbouras13/plassembler
- name: Mash
  role: Computes MinHash distances between each assembled plasmid contig and the PLSDB
    sketch database to find similar known plasmids
  repo: https://github.com/marbl/Mash
- name: Unicycler
  role: Produces the hybrid plasmid assembly from which circularity status of each
    contig is determined
  repo: https://github.com/rrwick/Unicycler
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:304a1f20bb398d780fb2a28c992df8b68854e9f576e66f3673073d40291d8327
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbouras13/plassembler
  tools:
  - Mash
  - PLSDB
  - Plassembler
  - Python
  - Unicycler
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# contig-circularity-annotation

## Summary

Annotates each assembled plasmid contig with its circularity status from the Unicycler assembly and its best PLSDB match, computed by screening contigs against the Plassembler PLSDB sketch database with mash and keeping the lowest-distance hit within the 0.9 similarity / 0.1 distance thresholds.

## When to use

Use when assembled plasmid contigs (e.g. Unicycler FASTA output from Plassembler) need per-contig circularity and known-plasmid similarity annotation against PLSDB, summarized into a plasmid_summary.tsv table.

## When NOT to use

- The contigs are not plasmid contigs from a Plassembler/Unicycler hybrid assembly and no circularity information from the Unicycler graph is available
- A full functional/gene annotation of plasmid contigs is needed rather than a nearest-known-plasmid (PLSDB) similarity call
- The analysis does not use the Plassembler database, so no PLSDB sketch database (from install_database.py) is available for mash screening

## Inputs

- Assembled plasmid contig FASTA (Unicycler output produced by Plassembler)
- PLSDB sketch (mash) database from the Plassembler database installed by install_database.py
- Unicycler assembly graph / assembly information for circularity determination

## Outputs

- plasmid_summary.tsv: tab-separated table with one row per assembled contig, including circularity status and PLSDB top-hit match columns

## How to apply

Load the assembled plasmid contigs (Unicycler output FASTA) and the PLSDB sketch database shipped with the Plassembler database, which is installed via install_database.py. Sketch each contig and run mash dist against the PLSDB sketch database to obtain pairwise MinHash distances for every contig-database pair. For each contig, filter hits with the Plassembler thresholds (similarity 0.9 / maximum mash distance 0.1) and retain the lowest-distance hit as the top PLSDB match. Separately determine circularity from the Unicycler assembly (complete circular plasmids vs. partial contigs). Write the per-contig circularity and PLSDB match columns into plasmid_summary.tsv so each assembled contig is one row with both annotations populated.

## Related tools

- **Plassembler** (Implements the mash screening against PLSDB, threshold filtering, circularity extraction from the Unicycler assembly, and writes plasmid_summary.tsv) — https://github.com/gbouras13/plassembler
- **Mash** (Computes MinHash distances between each assembled plasmid contig and the PLSDB sketch database to find similar known plasmids) — https://github.com/marbl/Mash
- **Unicycler** (Produces the hybrid plasmid assembly from which circularity status of each contig is determined) — https://github.com/rrwick/Unicycler

## Examples

```
mash dist assembled_plasmid_contigs.msh plsdb_sketch.msh > plsdb_hits.tsv
```

## Evaluation signals

- plasmid_summary.tsv exists and is tab-separated
- The table contains exactly one row per assembled plasmid contig
- Circularity and PLSDB match columns are populated for each reported contig
- Every reported PLSDB match satisfies the maximum mash distance threshold of 0.1 (similarity 0.9)
- Each contig's reported match is the lowest mash distance among its hits below the threshold

## Limitations

- Contigs with no PLSDB hit within the 0.1 mash distance threshold will not receive a PLSDB match annotation, so genuinely novel plasmids are left unmatched
- mash MinHash distances are an approximation of true sequence distance; the top hit is only the closest sketch-based neighbor below the threshold
- Linear plasmids can be misassembled by the Unicycler-based approach (e.g. the K. variicola linear plasmid recovered incomplete, a known issue), which can corrupt circularity annotation for such contigs

## Evidence

- [methods] mash screening of contigs against PLSDB: "`plassembler` uses [mash](https://github.com/marbl/Mash) as a quick way to determine whether each assembled contig has any similar hits in [PLSDB]"
- [discussion] Plassembler database and PLSDB mapping: "Adds install_database.py, the Plassembler database and functionality for mapping plasmid contigs to PLDSB."
- [abstract] hybrid assembly approach: "Plassembler was developed to provide a tool that automatically assembles and outputs bacterial plasmids using a hybrid assembly approach"
- [results] known issue of the Unicycler approach: "The only misassembly is of the linear plasmid in _K variicola_ , which is a known issue of the Unicycler approach"
