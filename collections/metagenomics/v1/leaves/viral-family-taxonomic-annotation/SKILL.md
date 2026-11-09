---
name: viral-family-taxonomic-annotation
description: Use when draft vMAGs (e.g., from ViralCC's VIRAL_BIN output directories)
  need family-level taxonomic labels to compute bin purity, or when reproducing the
  Caudovirales-family purity fractions reported for the human gut, cow fecal, and
  wastewater Hi-C datasets.
when_to_use_negative:
- Input is raw shotgun/Hi-C reads rather than assembled, binned vMAG FASTA contigs
  — viral identification (VirSorter) and binning (ViralCC pipeline) must happen first
- Goal is discriminating viral from bacterial/archaeal contigs — family-level classifiers
  are for classifying already-identified viral contigs, not detecting them (per Demovir's
  own scope note)
- 'TAXAassign is expected to fail because contigs are heavily novel: the article notes
  ''TAXAassign can hardly annotate viral contigs'', so low-coverage dark-matter vMAGs
  may yield no family assignment'
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: TAXAassign
  role: Performs the alignment-based family-level taxonomic assignment of vMAG contigs
    against a local blastn/nt database with NCBI Taxonomy resolution
  repo: https://github.com/umerijaz/TAXAassign
- name: ViralCC
  role: Produces the draft vMAG FASTA inputs via its VIRAL_BIN output directories
    that are then family-annotated
  repo: https://github.com/dyxstat/ViralCC
- name: Demovir
  role: Alternative democratic classifier that assigns viral contigs to Order/Family
    level by amino-acid-level gene comparison against the viral TrEMBL subset (archived;
    database out of date)
  repo: https://github.com/feargalr/Demovir
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:1791e8f12a0f92bc4bc7403c8b57739321576c4271198c4c4b44c826a7fd5501
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/umerijaz/TAXAassign
  tools:
  - Demovir
  - TAXAassign
  - ViralCC
  - pandas
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# viral-family-taxonomic-annotation

## Summary

Annotates draft vMAGs at the family level using a TAXAassign-style alignment-based assignment against a curated viral reference genome database, then tabulates per-dataset family-consistency (purity) fractions with pandas — reproducing ViralCC's family-level purity analysis (90.6% human gut, 82.8% cow fecal, 85.4% wastewater; Fig. 5).

## When to use

Use when draft vMAGs (e.g., from ViralCC's VIRAL_BIN output directories) need family-level taxonomic labels to compute bin purity, or when reproducing the Caudovirales-family purity fractions reported for the human gut, cow fecal, and wastewater Hi-C datasets.

## When NOT to use

- Input is raw shotgun/Hi-C reads rather than assembled, binned vMAG FASTA contigs — viral identification (VirSorter) and binning (ViralCC pipeline) must happen first
- Goal is discriminating viral from bacterial/archaeal contigs — family-level classifiers are for classifying already-identified viral contigs, not detecting them (per Demovir's own scope note)
- TAXAassign is expected to fail because contigs are heavily novel: the article notes 'TAXAassign can hardly annotate viral contigs', so low-coverage dark-matter vMAGs may yield no family assignment

## Inputs

- Draft vMAG FASTA sequences from ViralCC VIRAL_BIN output directories (human gut, cow fecal, wastewater datasets)
- Curated viral reference genome database (e.g., NCBI nt / RefSeq viral genomes) installed for local blastn
- NCBI Taxonomy loaded into a BioSQL MySQL or sqlite3 database (per TAXAassign setup)

## Outputs

- Per-vMAG family-level taxonomic assignments for each dataset
- family_annotation_purity_summary.csv containing annotation results, family-level purity fractions (e.g., 173/191=90.6%), and per-family composition breakdowns

## How to apply

Collect the draft vMAG FASTA sequences from the ViralCC VIRAL_BIN output directories for each dataset (expected totals: 191 human gut, 320 cow fecal, 693 wastewater vMAGs). Assign family-level taxonomy to every vMAG with a TAXAassign-style pipeline: blastn (Blastn 2.28+ or newer, since only 2.28+ emits Taxa IDs per hit) against a curated viral reference genome database, with parameters such as MINIMUM_PERCENT_IDENT=97, MINIMUM_QUERY_COVERAGE=97, and CONSENSUS_THRESHOLD=90, resolving lineage via NCBI Taxonomy stored in a BioSQL MySQL/sqlite3 database. Tabulate each vMAG's family assignment per dataset with pandas, then define a vMAG as pure when all member contigs share one family-level assignment (dominated by the Caudovirales families Myoviridae, Siphoviridae, and Podoviridae). Recompute purity as consistent-assignment counts divided by total vMAGs per dataset and check against the reported fractions: 173/191 (90.6%) human gut, 265/320 (82.8%) cow fecal, 592/693 (85.4%) wastewater. Write the per-family composition, purity, and annotation tables to family_annotation_purity_summary.csv.

## Related tools

- **TAXAassign** (Performs the alignment-based family-level taxonomic assignment of vMAG contigs against a local blastn/nt database with NCBI Taxonomy resolution) — https://github.com/umerijaz/TAXAassign
- **ViralCC** (Produces the draft vMAG FASTA inputs via its VIRAL_BIN output directories that are then family-annotated) — https://github.com/dyxstat/ViralCC
- **Demovir** (Alternative democratic classifier that assigns viral contigs to Order/Family level by amino-acid-level gene comparison against the viral TrEMBL subset (archived; database out of date)) — https://github.com/feargalr/Demovir

## Examples

```
import pandas as pd; df = pd.DataFrame({'consistent':[173,265,592],'total':[191,320,693]}, index=['human_gut','cow_fecal','wastewater']); (df.consistent/df.total).to_csv('family_annotation_purity_summary.csv')
```

## Evaluation signals

- Recomputed purity fractions match the reported values exactly: 173/191 (90.6%) human gut, 265/320 (82.8%) cow fecal, 592/693 (85.4%) wastewater
- Denominators equal the expected vMAG counts per dataset (191, 320, 693) recovered from the ViralCC VIRAL_BIN directories
- Dominant assignments are Caudovirales families Myoviridae, Siphoviridae, and Podoviridae across all three datasets, as in Fig. 5
- family_annotation_purity_summary.csv exists with per-dataset annotation, purity, and per-family composition fields populated via pandas
- Every vMAG in each dataset receives a family-level assignment (or is explicitly recorded as unassigned) before purity is tabulated

## Limitations

- TAXAassign can hardly annotate viral contigs — the article notes this exact weakness disabled HiCBin's viral binning, so expect sparse assignments for divergent vMAGs
- Family assignments are reference-dependent: viruses without homologous reference genomes (viral dark matter) cannot be classified, biasing purity toward well-characterized Caudovirales families
- TAXAassign setup is heavy: requires Blastn 2.28+ with a locally installed nt database, GNU Parallel, and a BioSQL MySQL/sqlite3 taxonomy database; the nt database and NCBI links must be kept current

## Evidence

- [intro] HiCBin requires the taxonomic annotation of some contigs by TAXAassign (https://github.com/umerijaz/TAXAassign) to generate the intra-species contacts: "HiCBin requires the taxonomic annotation of some contigs by TAXAassign (https://github.com/umerijaz/TAXAassign) to generate the intra-species contacts"
- [readme] TAXAassign is useful for annotating nucleotide sequences (contigs from assemblies, reads from whole-shot gun sequencing, 16S rRNA sequences, etc.) at different taxonomic levels: [evidence span withheld — anchor status: external_doc]
- [methods] python ./viralcc.py pipeline [Parameters] FASTA_file BAM_file VIRAL_file OUTPUT_directory: "python ./viralcc.py pipeline [Parameters] FASTA_file BAM_file VIRAL_file OUTPUT_directory"
- [abstract] ViralCC can also reveal the taxonomic structure of viruses and virus-host pairs in microbial communities.: "ViralCC can also reveal the taxonomic structure of viruses and virus-host pairs in microbial communities."
- [readme] The current version of TAXAassign only works with Blastn 2.28+ as previous versions dont give Taxa IDs in output format.: [evidence span withheld — anchor status: external_doc]
