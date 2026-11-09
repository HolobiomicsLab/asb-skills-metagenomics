---
name: family-composition-tabulation-across-datasets
description: Use when you have draft vMAG FASTA files from ViralCC VIRAL_BIN output
  for multiple datasets and need to quantify family-level purity (single-family vs
  mixed-family bins) and per-family composition.
when_to_use_negative:
- Input is raw shotgun/Hi-C FASTQ or unfiltered contigs rather than ViralCC-recovered
  draft vMAGs — purity of un-binned contigs is not defined
- The goal is completeness/contamination assessment — the paper notes CheckV 'is unable
  to estimate the contamination of vMAGs' and this skill only measures family-level
  purity, not marker-gene-based quality
- Taxonomic classification of individual contigs to distinguish viral from bacterial/archaeal
  sequence — Demovir-style family/order classifiers are explicitly not suited for
  that
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: ViralCC
  role: Produces the draft vMAG FASTA files (VIRAL_BIN output) whose family composition
    and purity are tabulated
  repo: https://github.com/dyxstat/ViralCC
- name: TAXAassign
  role: Alignment-based family-level taxonomic assignment of vMAG contigs via blastn
    against NCBI nt with Taxonomy resolution
  repo: https://github.com/umerijaz/TAXAassign
- name: pandas
  role: Tabulates annotation, purity fractions, and per-family composition into family_annotation_purity_summary.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:a61ea82f34ad370cc057f000eea77d1a162b52d2876e6b15114dd97f65d5958e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - TAXAassign
  - ViralCC
  - pandas
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# family-composition-tabulation-across-datasets

## Summary

Reproduces the family-level purity statistics of ViralCC-recovered vMAGs (Fig. 5 of the ViralCC paper) by taxonomically annotating every vMAG from the human gut, cow fecal, and wastewater datasets and tabulating how many vMAGs contain contigs from a single viral family.

## When to use

Use when you have draft vMAG FASTA files from ViralCC VIRAL_BIN output for multiple datasets and need to quantify family-level purity (single-family vs mixed-family bins) and per-family composition.

## When NOT to use

- Input is raw shotgun/Hi-C FASTQ or unfiltered contigs rather than ViralCC-recovered draft vMAGs — purity of un-binned contigs is not defined
- The goal is completeness/contamination assessment — the paper notes CheckV 'is unable to estimate the contamination of vMAGs' and this skill only measures family-level purity, not marker-gene-based quality
- Taxonomic classification of individual contigs to distinguish viral from bacterial/archaeal sequence — Demovir-style family/order classifiers are explicitly not suited for that

## Inputs

- Draft vMAG FASTA files from ViralCC VIRAL_BIN output directories (human gut, cow fecal, wastewater datasets)
- Curated viral reference genome database for TAXAassign-style family-level annotation
- NCBI nt BLAST database and NCBI Taxonomy (sqlite3/MySQL BioSQL or blastn-provided taxids)

## Outputs

- family_annotation_purity_summary.csv with per-vMAG family assignments, purity fractions (e.g. 90.6%, 82.8%, 85.4%), and per-family composition per dataset

## How to apply

Collect the draft vMAG FASTA sequences from the ViralCC VIRAL_BIN output directories for each dataset (human gut, cow fecal, wastewater). Assign family-level taxonomy to each vMAG by alignment-based annotation of its contigs against a curated viral reference database using a TAXAassign-style assignment (blastn against nt with NCBI Taxonomy resolution, with minimum percent identity, query coverage, and consensus thresholds). Tabulate per-dataset the family assignment of every vMAG and count vMAGs whose member contigs share one consistent family-level assignment. Recompute purity fractions as consistent-assignment counts over total vMAGs per dataset and verify against the reported 173/191 (90.6%) human gut, 265/320 (82.8%) cow fecal, and 592/693 (85.4%) wastewater. Write the annotation, purity, and per-family composition breakdown (Caudovirales families Myoviridae, Siphoviridae, Podoviridae dominate) to family_annotation_purity_summary.csv using pandas.

## Related tools

- **ViralCC** (Produces the draft vMAG FASTA files (VIRAL_BIN output) whose family composition and purity are tabulated) — https://github.com/dyxstat/ViralCC
- **TAXAassign** (Alignment-based family-level taxonomic assignment of vMAG contigs via blastn against NCBI nt with Taxonomy resolution) — https://github.com/umerijaz/TAXAassign
- **pandas** (Tabulates annotation, purity fractions, and per-family composition into family_annotation_purity_summary.csv)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('family_annotation_purity_summary.csv'); print(df.groupby('dataset')['single_family'].mean())"
```

## Evaluation signals

- Recomputed purity fractions match the reported values: 173/191 (90.6%) human gut, 265/320 (82.8%) cow fecal, 592/693 (85.4%) wastewater vMAGs with single-family membership
- Per-vMAG family assignment table has one family label per vMAG and every vMAG in each dataset is accounted for (denominators 191, 320, 693)
- Per-family composition is dominated by Caudovirales families (Myoviridae, Siphoviridae, Podoviridae) in all three samples, consistent with Fig. 5
- TAXAassign thresholds (minimum percent identity, minimum query coverage, consensus threshold, e.g. defaults 97/97/90 in TAXAassign.sh) are recorded and applied uniformly across datasets
- family_annotation_purity_summary.csv is well-formed with annotation, purity, and composition columns and loads cleanly via pandas

## Limitations

- Family-level purity measures consistency of taxonomic assignment, not contamination in the CheckM sense; CheckV cannot estimate vMAG contamination since no universal marker gene set exists for viruses
- TAXAassign is noted as barely able to annotate viral contigs in general (HiCBin fails to bin viral contigs for this reason), so annotation against a curated viral reference database is required
- TAXAassign depends on a local NCBI nt BLAST database, Blastn 2.28+, BioSQL-hosted NCBI Taxonomy (MySQL or sqlite3), and GNU Parallel; taxonomic databases must be kept updated

## Evidence

- [abstract] ViralCC can also reveal the taxonomic structure of viruses and virus-host pairs in microbial communities.: "ViralCC can also reveal the taxonomic structure of viruses and virus-host pairs in microbial communities."
- [intro] TAXAassign used for taxonomic annotation: "HiCBin requires the taxonomic annotation of some contigs by TAXAassign (https://github.com/umerijaz/TAXAassign) to generate the intra-species contacts"
- [readme] TAXAassign parameters in README: "NUMBER_OF_REFERENCE_MATCHES=10
MINIMUM_PERCENT_IDENT=97
MINIMUM_QUERY_COVERAGE=97
CONSENSUS_THRESHOLD=90"
- [intro] CheckV contamination limitation: "CheckV is unable to estimate the contamination of vMAGs since there is no such marker gene set available for viruses21"
- [readme] Reproduction scripts pointer: "Scripts to process the intermediate data and plot figures of our ViralCC paper are available [here](https://github.com/dyxstat/Reproduce_ViralCC/tree/main/Scripts)."
