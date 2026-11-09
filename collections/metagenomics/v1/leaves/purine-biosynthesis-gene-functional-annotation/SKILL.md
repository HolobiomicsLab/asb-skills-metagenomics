---
name: purine-biosynthesis-gene-functional-annotation
description: Use when a clade of MAGs/SGBs (e.g. Saccharofermentanales SGB009, SGB034,
  SGB080) may lack a conserved set of universal single-copy orthologs and you need
  to identify which missing USCO genes map to a specific pathway such as de novo purine
  biosynthesis.
when_to_use_negative:
- Input genomes are already provided as a BUSCO full_table.tsv from a non-bacteria_odb10
  lineage — the 124-USCO prokaryote set is lineage-specific and re-running with a
  different database will not reproduce the matrix
- The goal is de novo pathway reconstruction of a single genome rather than detecting
  clade-wide absence of universal single-copy orthologs across multiple genomes
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0085
tools:
- name: BUSCO
  role: Run in genome mode (-m genome, -q) against bacteria_odb10 to classify each
    of the 124 prokaryotic USCOs as single-copy, duplicated, fragmented or missing
    per Saccharofermentanales genome
- name: KEGG
  role: Functional category annotation used to flag universally absent USCO genes
    involved in de novo purine biosynthesis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:b6ef2f309c075a944d648b21ae6efc2060fdd8a7eaa2921cc2de1da830fe26f5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSCO
  - KEGG
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# purine-biosynthesis-gene-functional-annotation

## Summary

Screens Saccharofermentanales (and other VMGC SGB) genomes for missing universal single-copy orthologs (USCOs) using BUSCO bacteria_odb10 (124 USCOs), then functionally annotates the universally absent USCOs via KEGG to flag those involved in de novo purine biosynthesis.

## When to use

Use when a clade of MAGs/SGBs (e.g. Saccharofermentanales SGB009, SGB034, SGB080) may lack a conserved set of universal single-copy orthologs and you need to identify which missing USCO genes map to a specific pathway such as de novo purine biosynthesis.

## When NOT to use

- Input genomes are already provided as a BUSCO full_table.tsv from a non-bacteria_odb10 lineage — the 124-USCO prokaryote set is lineage-specific and re-running with a different database will not reproduce the matrix
- The goal is de novo pathway reconstruction of a single genome rather than detecting clade-wide absence of universal single-copy orthologs across multiple genomes

## Inputs

- Saccharofermentanales genome FASTA files (SGB009, SGB034, SGB080 and related MAGs) from VMGC_prokaryote_MAG.tar.gz / VMGC_prokaryote_SGB.tar.gz (Zenodo record 10457006)
- BUSCO 5.4.2 with the bacteria_odb10 lineage database (124 prokaryotic USCOs)
- Per-run full_table.tsv BUSCO output files

## Outputs

- Genome-by-USCO presence/absence matrix across all Saccharofermentanales genomes
- List of USCOs absent in every genome of the clade (expected: nine)
- Functional (KEGG) annotation of the absent USCOs identifying those in de novo purine biosynthesis (expected: five)

## How to apply

Retrieve the target Saccharofermentanales genome FASTAs (SGB009, SGB034, SGB080 and related MAGs) from the VMGC Zenodo deposit 10457006 and run BUSCO 5.4.2 in genome mode (-m genome, quiet -q) against the bacteria_odb10 lineage, which contains the 124 universal single-copy orthologs for prokaryotes. Parse each run's full_table.tsv to classify each of the 124 USCOs as single-copy, duplicated, fragmented or missing per genome, then assemble a genome-by-USCO presence/absence matrix encoding complete single-copy USCOs as present and fragmented/missing USCOs as absent. Identify USCOs absent in every genome of the clade and annotate those by functional category (e.g. KEGG pathways) to flag enzymes involved in de novo purine biosynthesis. The expected pattern is nine USCOs absent in all Saccharofermentanales genomes, five of which encode de novo purine biosynthesis enzymes.

## Related tools

- **BUSCO** (Run in genome mode (-m genome, -q) against bacteria_odb10 to classify each of the 124 prokaryotic USCOs as single-copy, duplicated, fragmented or missing per Saccharofermentanales genome)
- **KEGG** (Functional category annotation used to flag universally absent USCO genes involved in de novo purine biosynthesis)

## Examples

```
busco -m genome -q -i SGB009.fa -o SGB009_busco --lineage_dataset bacteria_odb10
```

## Evaluation signals

- The genome-by-USCO matrix reproduces nine USCOs absent in all Saccharofermentanales species (SGB009, SGB034, SGB080)
- Five of the nine universally absent USCOs are annotated as de novo purine biosynthesis enzymes, consistent with Fig. 3c and Extended Data Fig. 4c
- Each of the 124 USCOs in bacteria_odb10 is assigned exactly one state (single-copy, duplicated, fragmented or missing) per genome in the parsed full_table.tsv
- Matrix row/column counts match the number of Saccharofermentanales genomes analysed and the 124 USCOs

## Limitations

- BUSCO/CheckM2-style completeness estimates can be biased; the paper notes 'the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm'
- Encoding fragmented USCOs as absent treats partial gene retention as loss, which can overstate pathway deficiency
- Functional assignment depends on KEGG coverage; the paper reports only 67.4% of VMGC protein-coding genes could be accommodated in KEGG

## Evidence

- [abstract] Similar assessment results were also revealed by other algorithms such as CheckM and BUSCO (based on 124 universal single-copy orthologs (USCOs) for prokaryotes32): "Similar assessment results were also revealed by other algorithms such as CheckM and BUSCO (based on 124 universal single-copy orthologs (USCOs) for prokaryotes32)"
- [readme] The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006, including the data presented in the table below."
