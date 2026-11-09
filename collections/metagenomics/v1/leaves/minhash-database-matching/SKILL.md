---
name: minhash-database-matching
description: Use when assembled plasmid contigs (e.g. Unicycler output FASTA from
  Plassembler) need to be matched against a known plasmid database to annotate similarity
  to PLSDB entries.
when_to_use_negative:
- Input is raw long/short reads rather than assembled plasmid contigs (read removal
  and assembly are handled by earlier Plassembler steps, not this Mash-based matching)
- Goal is functional/gene-level plasmid annotation rather than database-level similarity
  to known plasmids (Mash gives MinHash distance, not gene content)
- No Plassembler/PLSDB sketch database has been installed (install_database.py has
  not been run), so no reference sketches are available
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Mash
  role: Sketches each plasmid contig and computes MinHash distances (mash dist) of
    every contig against the PLSDB sketch database
  repo: https://github.com/marbl/Mash
- name: PLSDB
  role: Reference plasmid database of known plasmid sketches that assembled contigs
    are screened against
- name: Plassembler
  role: Implements the distance filtering (similarity 0.9 / distance 0.1), top-hit
    selection, and writes plasmid_summary.tsv; supplies the PLSDB sketch database
    via install_database.py
  repo: https://github.com/gbouras13/plassembler
- name: Unicycler
  role: Produces the assembled plasmid contigs and the circularity status annotated
    per contig
  repo: https://github.com/rrwick/Unicycler
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:d2e6d7187614e2cb0c5a6f100a837035da951fd18b8c073bab035610258f74e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marbl/Mash
  tools:
  - Mash
  - PLSDB
  - Plassembler
  - Python
  - Unicycler
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# minhash-database-matching

## Summary

Sketch assembled plasmid contigs with Mash and screen them against the PLSDB sketch database shipped with the Plassembler database (install_database.py) to identify which contigs match known plasmids, keeping the lowest-distance top hit per contig under a 0.1 mash distance threshold.

## When to use

Use when assembled plasmid contigs (e.g. Unicycler output FASTA from Plassembler) need to be matched against a known plasmid database to annotate similarity to PLSDB entries.

## When NOT to use

- Input is raw long/short reads rather than assembled plasmid contigs (read removal and assembly are handled by earlier Plassembler steps, not this Mash-based matching)
- Goal is functional/gene-level plasmid annotation rather than database-level similarity to known plasmids (Mash gives MinHash distance, not gene content)
- No Plassembler/PLSDB sketch database has been installed (install_database.py has not been run), so no reference sketches are available

## Inputs

- Assembled plasmid contigs FASTA (Unicycler output from Plassembler)
- PLSDB sketch database shipped with the Plassembler database (installed by install_database.py)

## Outputs

- Per-contig PLSDB top hit (lowest mash distance match) with distance value
- Per-contig circularity status from the Unicycler assembly
- Summary table plasmid_summary.tsv (tab-separated, one row per assembled contig)

## How to apply

Load the assembled plasmid contigs (Unicycler output FASTA) and the PLSDB sketch database installed via Plassembler's install_database.py. Sketch each plasmid contig and run mash dist against the PLSDB sketch database to compute pairwise MinHash distances for every contig-database pair. For each contig, consider all matches below the Plassembler distance thresholds (similarity 0.9 / maximum mash distance 0.1) and keep the single PLSDB match with the lowest mash distance as the top hit. Annotate each contig with its circularity status as determined from the Unicycler assembly, and write all per-contig circularity and PLSDB match columns to a summary table plasmid_summary.tsv.

## Related tools

- **Mash** (Sketches each plasmid contig and computes MinHash distances (mash dist) of every contig against the PLSDB sketch database) — https://github.com/marbl/Mash
- **PLSDB** (Reference plasmid database of known plasmid sketches that assembled contigs are screened against)
- **Plassembler** (Implements the distance filtering (similarity 0.9 / distance 0.1), top-hit selection, and writes plasmid_summary.tsv; supplies the PLSDB sketch database via install_database.py) — https://github.com/gbouras13/plassembler
- **Unicycler** (Produces the assembled plasmid contigs and the circularity status annotated per contig) — https://github.com/rrwick/Unicycler

## Examples

```
mash dist plasmid_contigs.fasta PLSDB.msh > plsdb_mash_dist.tsv
```

## Evaluation signals

- plasmid_summary.tsv exists and is tab-separated
- plasmid_summary.tsv contains one row per assembled plasmid contig
- Circularity and PLSDB match columns are populated for every contig row
- Each reported PLSDB match satisfies the maximum mash distance threshold of 0.1 (equivalently similarity >= 0.9)
- For each contig, the reported match is the lowest mash distance among all PLSDB hits below the threshold

## Limitations

- Matches are MinHash-based sequence similarity only; contigs with no PLSDB hit below distance 0.1 are not matched, so novel plasmids remain unannotated
- Only the single lowest-distance PLSDB hit per contig is kept, so additional sub-threshold matches are discarded

## Evidence

- [methods] mash screening of contigs against PLSDB: "`plassembler` uses [mash](https://github.com/marbl/Mash) as a quick way to determine whether each assembled contig has any similar hits in [PLSDB]"
- [discussion] Plassembler database installation: "Adds install_database.py, the Plassembler database and functionality for mapping plasmid contigs to PLDSB."
