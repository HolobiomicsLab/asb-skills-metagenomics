---
name: metagenomic-read-mapping-quantification
description: Use when short-read shotgun metagenomes (FASTQ) must be converted from
  mapped-read counts into genome-equivalent relative abundances (cell fractions) against
  a source-associated genome database for source tracking.
when_to_use_negative:
- Input data are amplicon (e.g., 16S rRNA) reads rather than short-read shotgun metagenomes,
  since the framework relies on genome-wide read mapping metrics
- A mass-based (e.g., mass-balance of mass inputs) apportionment is required, because
  SourceApp portions estimate proportions of cells based on genome equivalents, not
  mass
- No source-associated genome database exists for the sources of interest and none
  can be curated, since quantification depends on genome-to-source associations
edam_operation: http://edamontology.org/operation_3230
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: CoverM
  role: Maps metagenomic reads to the source-associated genome databases and computes
    per-genome coverage/relative abundance (v0.7.0)
  repo: https://github.com/wwood/CoverM
- name: MicrobeCensus
  role: Estimates average genome size per metagenome to compute total genome equivalents
    (GEQ) for cell-fraction normalization (bundled as MicrobeCensus-SourceApp 1.1.2)
- name: SourceApp
  role: Integrates mapping filters and GEQ normalization (--use-geq) to output source
    cell fractions for attribution and apportionment
  repo: https://github.com/bglindner/SourceApp
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:4878872f530390490792b5414712b5dabd62581835e2a005f67c3b81e53131a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wwood/CoverM
  tools:
  - CoverM
  - MicrobeCensus
  - Python
  - SourceApp
  edam_operation: http://edamontology.org/operation_3230
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metagenomic-read-mapping-quantification

## Summary

Quantifies short-read shotgun metagenomes by mapping them to source-associated genome databases with CoverM (v0.7.0) and normalizing mapped sequence depth by genome equivalents (GEQ, estimated with MicrobeCensus) to obtain cell-fraction-style relative abundances used by SourceApp for fecal source attribution and apportionment.

## When to use

Use when short-read shotgun metagenomes (FASTQ) must be converted from mapped-read counts into genome-equivalent relative abundances (cell fractions) against a source-associated genome database for source tracking.

## When NOT to use

- Input data are amplicon (e.g., 16S rRNA) reads rather than short-read shotgun metagenomes, since the framework relies on genome-wide read mapping metrics
- A mass-based (e.g., mass-balance of mass inputs) apportionment is required, because SourceApp portions estimate proportions of cells based on genome equivalents, not mass
- No source-associated genome database exists for the sources of interest and none can be curated, since quantification depends on genome-to-source associations

## Inputs

- Gzipped short-read shotgun metagenomic FASTQ files (e.g., 2 x 150-bp Illumina NovaSeq 6000 reads from mesocosm or waterway samples)
- Source-associated reference genome database (FASTA genomes with known source associations, e.g., SourceApp default database from Zenodo)
- CoverM BAM files sorted by reference (optional alternative input to raw reads)
- Per-genome mapped read counts / sequence depth table

## Outputs

- Per-genome sequence depth / mapped read counts table per metagenome
- Estimated average genome size per sample from MicrobeCensus
- Total genome equivalents (GEQ) per sample
- cell_fractions.csv with Source and Fraction columns (genome-copy-based relative abundance per source category)

## How to apply

Load per-genome mapped read counts and sequence depth for each mesocosm metagenome as quantified by CoverM (v0.7.0) against the source-associated genome databases, respecting the framework's mapping filters (93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0.01% relative-abundance limit of detection). Run MicrobeCensus (MicrobeCensus-SourceApp 1.1.2) on each metagenome to estimate average genome size and compute the total genome equivalents (GEQ) of the sample. For each source category j, sum the sequence depth X of all genomes belonging to that source (including cross-reactive genomes flagged at ANI >= 95%) and divide by GEQ following the cell fraction formula cellfraction_j = sum(X_i)/GEQ. Assemble per-source cell fractions into a table with Source and Fraction columns mirroring SourceApp's fractioning output; in SourceApp itself this GEQ-based normalization is engaged via the --use-geq flag.

## Related tools

- **CoverM** (Maps metagenomic reads to the source-associated genome databases and computes per-genome coverage/relative abundance (v0.7.0)) — https://github.com/wwood/CoverM
- **MicrobeCensus** (Estimates average genome size per metagenome to compute total genome equivalents (GEQ) for cell-fraction normalization (bundled as MicrobeCensus-SourceApp 1.1.2))
- **SourceApp** (Integrates mapping filters and GEQ normalization (--use-geq) to output source cell fractions for attribution and apportionment) — https://github.com/bglindner/SourceApp

## Examples

```
coverm genome -r combined_genomes.fna -1 read1.fq -2 read2.fq --min-percent-identity 0.93 --min-aligned-percent 0.7 --methods mean
```

## Evaluation signals

- cell_fractions.csv exists and contains the Source and Fraction fields
- All fraction values fall within [0, 1]
- Every source represented in the genome database appears as a row in the output table
- Fractions for known spike-in sources in ground-truth mesocosms (Table S1) are detectable above the 0.01% limit of detection
- Per-source fractions sum to values consistent with genome-equivalent (not mass-balance) normalization, with sequence breadth detection threshold applied per genome

## Limitations

- SourceApp's reported portions are estimates of the proportions of cells belonging to each source category based on genome equivalents, not the results of a mass balance of mass inputs, and may vary from mass-based expectations
- Performance varied substantially by source; cows were underestimated across all samples due to undersampling of cow fecal microbiomes and poor understanding of cow fecal prokaryotic genomic content
- Substantial residuals remained after linear correction of cell fraction estimates even with the best performing parameters (r2 = 0.64)
- Sources below the 0.01% cell-fraction limit of detection are missed; species-level marker genomes have largely unknown persistence in the environment

## Evidence

- [intro] framework normalization rationale: "we theorized and tested a new framework for metagenomic FST based on read mapping of metagenomic data to source-associated genome databases and normalization by meaningful abundance units"
- [methods] CoverM version: "CoverM (v0.7.0; https://github.com/wwood/CoverM)"
- [results] genome-equivalents vs mass balance: "efforts to estimate the proportions of cells belonging to source categories based on genome equivalents"
- [readme] SourceApp --use-geq cell fraction output: "SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called) and has been benchmarked in laboratory trials"
- [readme] CoverM purpose: [evidence span withheld — anchor status: external_doc]
