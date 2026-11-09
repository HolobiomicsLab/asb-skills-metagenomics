---
name: prokaryotic-genome-completeness-assessment
description: Use when you have prokaryotic MAG FASTA files (e.g. VMGC SGBs) and need
  to classify each universal single-copy ortholog as single-copy, duplicated, fragmented
  or missing, and detect pathway-level absences such as de novo purine biosynthesis
  genes.
when_to_use_negative:
- Input is a non-redundant gene catalogue (e.g. VMGC-50/90/95) or viral/fungal genomes
  — USCO-based assessment with bacteria_odb10 applies to prokaryotic genomes only
- You only need overall quality tiers (medium/high/near-complete) already assigned
  via MIMAG-revised criteria rather than per-gene presence/absence of USCOs
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: BUSCO
  role: Run in genome mode (-m genome) against bacteria_odb10 to classify the 124
    USCOs as single-copy, duplicated, fragmented or missing per MAG
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:8ba5cf7cce52c17805422395cffd842c47a0a6bec81d12a444c43b83d2981b4d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSCO
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# prokaryotic-genome-completeness-assessment

## Summary

Assess MAG quality using BUSCO's 124 universal single-copy orthologs (USCOs) for prokaryotes to produce genome-by-USCO presence/absence matrices that expose systematically missing pathways, as done for Saccharofermentanales genomes in the VMGC.

## When to use

Use when you have prokaryotic MAG FASTA files (e.g. VMGC SGBs) and need to classify each universal single-copy ortholog as single-copy, duplicated, fragmented or missing, and detect pathway-level absences such as de novo purine biosynthesis genes.

## When NOT to use

- Input is a non-redundant gene catalogue (e.g. VMGC-50/90/95) or viral/fungal genomes — USCO-based assessment with bacteria_odb10 applies to prokaryotic genomes only
- You only need overall quality tiers (medium/high/near-complete) already assigned via MIMAG-revised criteria rather than per-gene presence/absence of USCOs

## Inputs

- Prokaryotic MAG genome FASTA files (e.g. VMGC SGB009, SGB034, SGB080 from Zenodo record 10457006)
- BUSCO bacteria_odb10 lineage database (124 USCOs for prokaryotes)

## Outputs

- Per-genome full_table.tsv from each BUSCO run classifying 124 USCOs as single-copy, duplicated, fragmented or missing
- Genome-by-USCO presence/absence matrix across target genomes
- List of USCO genes absent in all target genomes with functional (KEGG pathway) annotations

## How to apply

Retrieve the target genome FASTA files (e.g. SGB009, SGB034, SGB080 and related Saccharofermentanales MAGs from VMGC_prokaryote_MAG.tar.gz on Zenodo record 10457006). Run BUSCO 5.4.2 in genome mode (-m genome, quiet mode -q) against the bacteria_odb10 lineage database, which contains the 124 USCOs for prokaryotes. Parse each run's full_table.tsv to classify every USCO as single-copy, duplicated, fragmented or missing per genome, then assemble a genome-by-USCO presence/absence matrix encoding complete single-copy USCOs as present and fragmented/missing USCOs as absent. Identify USCOs absent in every target genome and annotate them by functional category (e.g. KEGG pathways) to flag metabolic absences; in the paper this revealed nine USCO genes absent in all Saccharofermentanales, five involved in de novo purine biosynthesis (consistent with Fig. 3c and Extended Data Fig. 4c).

## Related tools

- **BUSCO** (Run in genome mode (-m genome) against bacteria_odb10 to classify the 124 USCOs as single-copy, duplicated, fragmented or missing per MAG)

## Examples

```
busco -i SGB009.fa -m genome -q -l bacteria_odb10 -o SGB009_busco
```

## Evaluation signals

- Each genome's full_table.tsv accounts for exactly 124 USCOs with a status of Complete (single/duplicated), Fragmented or Missing
- The genome-by-USCO matrix reproduces the reported pattern of nine USCO genes absent in all Saccharofermentanales genomes (SGB009, SGB034, SGB080)
- Five of the nine all-absent USCOs annotate to de novo purine biosynthesis (Fig. 3c / Extended Data Fig. 4c pattern)
- Quality metrics agree with complementary algorithms such as CheckM on the same MAGs

## Limitations

- Completeness of some genomes may be underestimated by assessment algorithms, as noted for A. indicium and A. nucleatus with CheckM2
- Fragmented USCOs must be treated as absent for the presence/absence matrix, so partial gene retention can be conflated with true gene loss

## Evidence

- [abstract] Similar assessment results were also revealed by other algorithms such as CheckM and BUSCO (based on 124 universal single-copy orthologs (USCOs) for prokaryotes32): "Similar assessment results were also revealed by other algorithms such as CheckM and BUSCO (based on 124 universal single-copy orthologs (USCOs) for prokaryotes32)"
- [readme] The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006"
- [abstract] 10,127, 8,397 and 1,017 were classified as medium-quality, high-quality and near-complete genomes, respectively: "10,127, 8,397 and 1,017 were classified as medium-quality, high-quality and near-complete genomes, respectively"
