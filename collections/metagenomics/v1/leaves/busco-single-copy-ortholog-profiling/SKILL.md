---
name: busco-single-copy-ortholog-profiling
description: Use when you need to profile completeness of MAGs via universal single-copy
  orthologs, or detect systematic absence of USCO genes (e.g. purine biosynthesis)
  across a clade of genomes.
when_to_use_negative:
- Input is protein or transcript data rather than genome assemblies — BUSCO must be
  rerun with the appropriate mode instead of -m genome
- The goal is general genome quality scoring with CheckM2/CheckM-style completeness
  only, not per-gene USCO presence/absence inference
- Studying lineages whose USCO set differs from the bacteria_odb10 124-ortholog prokaryote
  set (e.g. eukaryotic or viral genomes)
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0622
tools:
- name: BUSCO
  role: Run in genome mode (-m genome) against bacteria_odb10 to assess presence of
    the 124 USCOs per genome
- name: KEGG
  role: Functional annotation of missing USCO genes to identify those involved in
    de novo purine biosynthesis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:f306b5b552545779d276f8bce94a6df6283915049db45e1efabe528d369cb837
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSCO
  - KEGG
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# busco-single-copy-ortholog-profiling

## Summary

Run BUSCO in genome mode against the bacteria_odb10 lineage (124 universal single-copy orthologs, USCOs, for prokaryotes) to build a genome-by-USCO presence/absence matrix, used in the VMGC paper to show that Saccharofermentanales genomes lack nine USCO genes, five of which encode de novo purine biosynthesis enzymes.

## When to use

Use when you need to profile completeness of MAGs via universal single-copy orthologs, or detect systematic absence of USCO genes (e.g. purine biosynthesis) across a clade of genomes.

## When NOT to use

- Input is protein or transcript data rather than genome assemblies — BUSCO must be rerun with the appropriate mode instead of -m genome
- The goal is general genome quality scoring with CheckM2/CheckM-style completeness only, not per-gene USCO presence/absence inference
- Studying lineages whose USCO set differs from the bacteria_odb10 124-ortholog prokaryote set (e.g. eukaryotic or viral genomes)

## Inputs

- Prokaryotic genome FASTA files (MAGs/SGBs, e.g. VMGC Saccharofermentanales SGB009, SGB034, SGB080 from VMGC_prokaryote_SGB.tar.gz, Zenodo record 10457006)
- BUSCO bacteria_odb10 lineage dataset (124 universal single-copy orthologs for prokaryotes)

## Outputs

- Per-genome BUSCO run output including full_table.tsv with per-USCO status (single-copy, duplicated, fragmented, missing)
- Genome-by-USCO presence/absence matrix across all genomes in the clade
- List of USCO genes absent in all clade genomes with functional annotations (e.g. KEGG pathway categories)

## How to apply

Retrieve the genome FASTA files of interest (e.g. SGB009, SGB034, SGB080 and related Saccharofermentanales MAGs from VMGC_prokaryote_SGB.tar.gz at Zenodo record 10457006). Run BUSCO 5.4.2 in genome mode (-m genome, quiet mode -q) against the bacteria_odb10 lineage database, which contains the 124 USCOs for prokaryotes. Parse each run's full_table.tsv to classify each USCO as single-copy, duplicated, fragmented or missing per genome, then assemble a genome-by-USCO presence/absence matrix, encoding complete single-copy USCOs as present and fragmented/missing USCOs as absent. Identify USCOs absent in every genome of the clade and annotate them by functional category (e.g. KEGG pathways) to flag involvement in de novo purine biosynthesis. The analysis is validated when the matrix reproduces the reported pattern of nine USCOs absent in all Saccharofermentanales, five tied to purine biosynthesis (Fig. 3c, Extended Data Fig. 4c).

## Related tools

- **BUSCO** (Run in genome mode (-m genome) against bacteria_odb10 to assess presence of the 124 USCOs per genome)
- **KEGG** (Functional annotation of missing USCO genes to identify those involved in de novo purine biosynthesis)

## Examples

```
busco -m genome -q --lineage_dataset bacteria_odb10 -i SGB009.fa -o busco_SGB009 --out_path busco_runs
```

## Evaluation signals

- Each genome's full_table.tsv accounts for all 124 bacteria_odb10 USCOs with a status of single-copy, duplicated, fragmented or missing
- Presence/absence matrix is complete: one row per genome (SGB009, SGB034, SGB080, related MAGs), one column per USCO
- Matrix reproduces the reported pattern: exactly nine USCO genes absent in all Saccharofermentanales species
- Five of the nine clade-wide missing USCOs annotate to KEGG de novo purine biosynthesis, consistent with Fig. 3c and Extended Data Fig. 4c
- Genomes expected to be complete (e.g. high-quality/near-complete VMGC MAGs) show the majority of USCOs present as single-copy, indicating the pipeline is not spuriously calling absences

## Limitations

- Completeness estimates from marker-based tools can be inaccurate for some taxa; the paper notes 'the completeness of A. indicium and A. nucleatus might be underestimated in the current CheckM2 algorithm', so USCO absence patterns may partly reflect genome quality rather than biology
- Fragmented USCOs are conservatively encoded as absent; genuinely present but assembly-fragmented genes will be misclassified
- Conclusions are tied to the bacteria_odb10 set of 124 USCOs; different lineage databases would change the gene set examined

## Evidence

- [abstract] Similar assessment results were also revealed by other algorithms such as CheckM and BUSCO (based on 124 universal single-copy orthologs (USCOs) for prokaryotes32): "Similar assessment results were also revealed by other algorithms such as CheckM and BUSCO (based on 124 universal single-copy orthologs (USCOs) for prokaryotes32)"
- [readme] The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006, including the data presented in the table below."
- [readme] Nonredundant prokaryotic genomes (n=786) | 440 MB | VMGC_prokaryote_SGB.tar.gz: "Nonredundant prokaryotic genomes (n=786) | 440 MB | VMGC_prokaryote_SGB.tar.gz"
