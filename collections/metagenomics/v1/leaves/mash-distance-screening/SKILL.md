---
name: mash-distance-screening
description: Use when you have assembled plasmid contigs (e.g. from Unicycler within
  Plassembler) and need to annotate each contig with its closest PLSDB plasmid match
  using Mash distance.
when_to_use_negative:
- Input is raw long/short reads rather than assembled plasmid contigs — mash screening
  here operates on Unicycler-assembled contigs after read extraction and hybrid assembly.
- You need full functional annotation or alignment-level comparison of plasmids rather
  than a fast MinHash similarity lookup — mash dist gives approximate distances, not
  alignments.
- Plasmid contigs are linear or from non-bacterial assemblies where PLSDB (a bacterial
  plasmid database) is not an appropriate reference space.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Mash
  role: Computes MinHash sketch distances (mash dist) between each assembled plasmid
    contig and the PLSDB sketch database to find similar known plasmids
  repo: https://github.com/marbl/Mash
- name: PLSDB
  role: Reference plasmid database screened with mash to determine whether each assembled
    contig matches a known plasmid
- name: Plassembler
  role: 'Implements the screening logic: filters PLSDB hits at similarity 0.9 / distance
    0.1, keeps the lowest-distance top hit, and writes plasmid_summary.tsv'
  repo: https://github.com/gbouras13/plassembler
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:c9fe570a6bf9a07141352eac7b359f8335b799848d3dd0bb119bd53b9f9920ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marbl/Mash
  tools:
  - Mash
  - PLSDB
  - Plassembler
  - Python
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mash-distance-screening

## Summary

Screens assembled plasmid contigs against the PLSDB sketch database with Mash MinHash distances to determine whether each contig matches a known plasmid, keeping the lowest-distance top hit under the Plassembler threshold of 0.1.

## When to use

Use when you have assembled plasmid contigs (e.g. from Unicycler within Plassembler) and need to annotate each contig with its closest PLSDB plasmid match using Mash distance.

## When NOT to use

- Input is raw long/short reads rather than assembled plasmid contigs — mash screening here operates on Unicycler-assembled contigs after read extraction and hybrid assembly.
- You need full functional annotation or alignment-level comparison of plasmids rather than a fast MinHash similarity lookup — mash dist gives approximate distances, not alignments.
- Plasmid contigs are linear or from non-bacterial assemblies where PLSDB (a bacterial plasmid database) is not an appropriate reference space.

## Inputs

- Assembled plasmid contigs in FASTA format (Unicycler output from the Plassembler pipeline)
- PLSDB sketch (mash) database shipped with the Plassembler database installed by install_database.py

## Outputs

- Per-contig PLSDB top-hit assignments with mash distances below the 0.1 threshold
- Per-contig circularity status derived from the Unicycler assembly
- Summary table plasmid_summary.tsv (tab-separated, one row per assembled contig)

## How to apply

Load the assembled plasmid contigs (Unicycler output FASTA) together with the PLSDB sketch database shipped with the Plassembler database (installed via install_database.py). Sketch each contig and run mash dist against the PLSDB sketches to compute pairwise MinHash distances for every contig–database pair; Plassembler uses mash 'as a quick way to determine whether each assembled contig has any similar hits in PLSDB'. Apply the Plassembler distance thresholds (similarity 0.9 / distance 0.1), discarding all matches above the maximum mash distance of 0.1, and for each contig retain only the PLSDB match with the lowest mash distance as the top hit. Combine the per-contig PLSDB match columns with circularity status derived from the Unicycler assembly and write everything to plasmid_summary.tsv.

## Related tools

- **Mash** (Computes MinHash sketch distances (mash dist) between each assembled plasmid contig and the PLSDB sketch database to find similar known plasmids) — https://github.com/marbl/Mash
- **PLSDB** (Reference plasmid database screened with mash to determine whether each assembled contig matches a known plasmid)
- **Plassembler** (Implements the screening logic: filters PLSDB hits at similarity 0.9 / distance 0.1, keeps the lowest-distance top hit, and writes plasmid_summary.tsv) — https://github.com/gbouras13/plassembler

## Examples

```
mash dist -t 0.1 assembled_plasmid_contigs.fasta PLSDB.msh > plsdb_hits.tsv
```

## Evaluation signals

- plasmid_summary.tsv exists and is tab-separated with one row per assembled contig
- circularity and PLSDB match columns are populated for each contig row
- every reported PLSDB match satisfies the maximum mash distance threshold of 0.1 (equivalently similarity >= 0.9)
- each contig carries at most one PLSDB top hit — the lowest-distance match among all hits passing the threshold
- the PLSDB sketch database was installed via install_database.py before screening

## Limitations

- Mash provides approximate MinHash distances, so the top hit is the closest PLSDB sketch under the 0.1 cutoff rather than an alignment-verified identity.
- PLSDB matching only annotates contigs against known plasmids; novel plasmids with no PLSDB neighbor below distance 0.1 will have no match.
- Circularity annotation depends on the Unicycler assembly; the known Unicycler limitation of misassembling linear plasmids (e.g. the 30408 bp vs 31780 bp K. variicola case) can affect downstream annotation.

## Evidence

- [methods] Plassembler uses mash as a quick way to determine whether each assembled contig has any similar hits in PLSDB: "`plassembler` uses [mash](https://github.com/marbl/Mash) as a quick way to determine whether each assembled contig has any similar hits in [PLSDB]"
- [discussion] Adds install_database.py, the Plassembler database and functionality for mapping plasmid contigs to PLDSB: "Adds install_database.py, the Plassembler database and functionality for mapping plasmid contigs to PLDSB."
- [other] Plassembler considers all PLSDB matches below a maximum mash distance threshold of 0.1 and keeps the lowest-distance top hit per contig: [evidence span withheld — anchor status: not_found]
- [other] For each contig, filter hits with the Plassembler distance thresholds (similarity 0.9 / distance 0.1) and keep the lowest-distance top hit: [evidence span withheld — anchor status: not_found]
