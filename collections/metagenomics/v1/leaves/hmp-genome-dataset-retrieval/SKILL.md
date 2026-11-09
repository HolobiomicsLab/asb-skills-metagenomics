---
name: hmp-genome-dataset-retrieval
description: Use when a benchmark requires the HMP prokaryotic genome set with trusted
  NCBI-taxonomy labels — e.g. evaluating a MAG taxonomy classifier where strain-level
  assignments are needed and GTDB-based tools cannot provide them.
when_to_use_negative:
- The benchmark targets GTDB-based evaluation or requires GTDB taxonomy natively —
  HMP labels are NCBI-based and would need conversion tables in the opposite direction.
- The question requires CAMI II mock community or real metagenomic sample data (e.g.
  GI-Illumina, SRR606249) rather than isolated reference genomes.
- Only MAGs from binning pipelines are available (e.g. MetaBAT2 bins) — this skill
  retrieves reference-grade complete genomes, not binned assemblies.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
tools:
- name: kMetaShot
  role: Runs on the retrieved HMP genome FASTA files against the HDF5 reference; its
    classifications (CO set) are compared against the extracted HMP NCBI labels (CE
    set)
  repo: https://github.com/gdefazio/kMetaShot
- name: GTDBtk
  role: Run on the same 939 HMP genomes (v1.0.2) for comparison; outputs converted
    from GTDB to NCBI taxonomy via ncbi_vs_gtdb_r89 tables
- name: conda
  role: Installs kMetaShot from bioconda to classify the retrieved genomes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:1ca2355fdbb5216f113a11cb9fbd1b15da7388af49c2e291123e45687203b1f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - GTDBtk
  - conda
  - kMetaShot
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# hmp-genome-dataset-retrieval

## Summary

Retrieves the 939 bacterial and archaeal Human Microbiome Project (HMP) genomes and their expected NCBI taxonomic labels from the NCBI Human Microbiome FTP site, forming the reference ground-truth set used to benchmark kMetaShot against GTDBtk and CAMITAX at strain, species, and genus levels.

## When to use

Use when a benchmark requires the HMP prokaryotic genome set with trusted NCBI-taxonomy labels — e.g. evaluating a MAG taxonomy classifier where strain-level assignments are needed and GTDB-based tools cannot provide them.

## When NOT to use

- The benchmark targets GTDB-based evaluation or requires GTDB taxonomy natively — HMP labels are NCBI-based and would need conversion tables in the opposite direction.
- The question requires CAMI II mock community or real metagenomic sample data (e.g. GI-Illumina, SRR606249) rather than isolated reference genomes.
- Only MAGs from binning pipelines are available (e.g. MetaBAT2 bins) — this skill retrieves reference-grade complete genomes, not binned assemblies.

## Inputs

- NCBI Human Microbiome FTP site (genome FASTA files and associated taxonomy flat files)
- kMetaShot HDF5 reference (e.g. kMetaShot_reference.h5, ~22 Gb, RefSeq 2022/07/31)
- ncbi_vs_gtdb_r89 conversion tables (https://data.gtdb.ecogenomic.org/releases/release89/89.0/) for cross-tool taxonomy harmonization

## Outputs

- 939 HMP genome FASTA files (934 strains, 556 species, 190 genera)
- Per-genome expected NCBI taxonomy labels (strain, species, genus) usable as ground truth (CE set)
- Classification outcome tables (CO, CT sets) feeding sensitivity/precision/FPR/BA/F1 computations

## How to apply

Download the 939 HMP genomes (937 bacteria and 2 archaea, spanning 190 genera, 556 species, and 934 strains) as FASTA files from the NCBI Human Microbiome FTP site. Extract expected taxonomic labels for each genome from the associated flat files on the FTP site; because HMP taxonomy is NCBI-based, these labels can be used directly with kMetaShot classifications without any taxonomy conversion, unlike GTDBtk outputs which require ncbi_vs_gtdb_r89 conversion tables. Treat the extracted labels as the 'expected' set (CE) in the paper's set-based metric definitions (Tp = CO∩CE, Fp = CO−(CO∩CE), Fn = CE−(CO∩CE), Tn = CT−(CO∩CE)) to compute sensitivity, precision, FPR, balanced accuracy, and F1-score at strain, species, and genus levels. Validate retrieval completeness against the paper's counts: 939 total genomes classified, 926 classified by kMetaShot, with only 11 strains and 2 species left unclassified.

## Related tools

- **kMetaShot** (Runs on the retrieved HMP genome FASTA files against the HDF5 reference; its classifications (CO set) are compared against the extracted HMP NCBI labels (CE set)) — https://github.com/gdefazio/kMetaShot
- **GTDBtk** (Run on the same 939 HMP genomes (v1.0.2) for comparison; outputs converted from GTDB to NCBI taxonomy via ncbi_vs_gtdb_r89 tables)
- **conda** (Installs kMetaShot from bioconda to classify the retrieved genomes)

## Examples

```
conda create --name kmetashot kmetashot=2.0=pyh7e72e81_1 python=3.9 -c bioconda && conda activate kmetashot && kMetaShot_classifier_NV.py -b hmp_genomes/ -r kMetaShot_reference.h5 -p 10 -o hmp_output
```

## Evaluation signals

- Genome count matches the paper's denominator: 939 HMP genomes (937 bacteria, 2 archaea) spanning 190 genera, 556 species, 934 strains.
- Taxonomy-label coverage: kMetaShot (NCBI-based) requires no taxonomy conversion; GTDBtk outputs must map cleanly through ncbi_vs_gtdb_r89 tables.
- Reproduced sensitivity lands near the reported values: ~82.0% (strain), ~89.2% (species), ~92.9% (genus), with correct assignments of 770/830/872 genomes.
- Balanced accuracy falls between 90.8% and 95.4%, and FPR between 0.2% (strains/species) and 1.9% (genus).
- Only 11 strains and 2 species remain unclassified, consistent with the paper's report on the HMP set.

## Limitations

- GTDBtk and CAMITAX provide no strain-level classification, so strain-level benchmarking is only possible for kMetaShot on this dataset.
- Taxonomy labels for HMP are NCBI-based; any comparator tool using GTDB taxonomy needs ncbi_vs_gtdb_r89 conversion, which can introduce mapping ambiguity.
- The kMetaShot reference is NCBI RefSeq-based (bacteria and archaea only), so classification is limited to prokaryotic taxa present in RefSeq.

## Evidence

- [results] The 939 (937 bacteria and 2 archaea) HMP genomes span 190 genera, 556 species, and 934 strains: "The 939 (937 bacteria and 2 archaea) HMP genomes span 190 genera, 556 species, and 934 strains"
- [methods] kMetaShot was tested on Bacterial and Archaeal HMP (Human Microbiome Project) genomes: "kMetaShot was tested on Bacterial and Archaeal HMP (Human Microbiome Project) [51] genomes"
- [methods] using the ncbi_vs_gtdb_r89 conversion tables: "using the ncbi_vs_gtdb_r89 conversion tables (available at https://data.gtdb.ecogenomic.org/releases/release89/89.0/)"
- [results] correctly assigned 770, 830, and 872 genomes at strain, species, and genus levels, respectively: "It correctly assigned 770, 830, and 872 genomes at strain, species, and genus levels, respectively"
- [results] We found that only 11 strains and 2 species were not classified at all.: "We found that only 11 strains and 2 species were not classified at all."
- [readme] kMetaShot reference represents prokaryotic RefSeq genomes and requires about 22Gb of storage.: "kMetaShot reference represents prokaryotic RefSeq genomes and requires about 22Gb of storage."
