---
name: mag-taxonomic-classification
description: Use when you have binned MAG FASTA files (from MetaBAT2 or similar) and
  need NCBI-taxonomy assignments up to strain level, comparing tools or benchmarking
  classification accuracy.
when_to_use_negative:
- Input is raw shotgun reads rather than binned MAG sequences — kMetaShot classifies
  assemblies/bins, not reads
- Available RAM cannot hold the ~22 Gb HDF5 reference (kMetaShot requires at least
  32 Gb to load); use a marker-gene tool instead
- Target taxa are outside the prokaryotic reference (bacteria/archaea RefSeq genomes);
  eukaryotic or viral MAGs are out of scope
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3174
tools:
- name: kMetaShot
  role: primary alignment-free MAG taxonomy classifier based on k-mer/minimizer counting
  repo: github.com/gdefazio/kMetaShot
- name: GTDBtk
  role: marker-gene-based comparator tool; outputs converted from GTDB to NCBI taxonomy
    via ncbi_vs_gtdb_r89 tables
- name: CAMITAX
  role: comparator tool run through Nextflow (v21.04.0) for benchmarking classification
    accuracy
- name: MetaBAT2
  role: bins contigs into MAGs prior to classification (--maxP 99 --minS 98 -m 1,500)
- name: Mash
  role: estimates pairwise ANI for strain-level assignment refinement (>=97% ANI)
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:241a4c9004c65334d7505a9c72672fcfa9d6f3eece843a71c6b4e13727dedf85
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMITAX
  - CMDbench
  - FastQC
  - GTDBtk
  - Mash
  - MegaHIT
  - MetaBAT2
  - Nextflow
  - Sickle
  - kMetaShot
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# mag-taxonomic-classification

## Summary

Taxonomically classify metagenome-assembled genomes (MAGs) using kMetaShot, an alignment-free k-mer/minimizer classifier benchmarked against GTDBtk and CAMITAX, with an ass2ref reliability filter to drop unreliable calls.

## When to use

Use when you have binned MAG FASTA files (from MetaBAT2 or similar) and need NCBI-taxonomy assignments up to strain level, comparing tools or benchmarking classification accuracy.

## When NOT to use

- Input is raw shotgun reads rather than binned MAG sequences — kMetaShot classifies assemblies/bins, not reads
- Available RAM cannot hold the ~22 Gb HDF5 reference (kMetaShot requires at least 32 Gb to load); use a marker-gene tool instead
- Target taxa are outside the prokaryotic reference (bacteria/archaea RefSeq genomes); eukaryotic or viral MAGs are out of scope

## Inputs

- Directory of MAG FASTA bin files (.fa/.fasta/.fna/.fa.gz/.fasta.gz/.fna.gz) or a multi-FASTA with one header per MAG
- kMetaShot HDF5 reference (e.g. kMetaShot_bacteria_archaea_2025-05-22.h5, ~22 Gb, built from RefSeq Release 213)
- For benchmarking: expected CAMI taxonomies or known community composition (e.g. 48 bacteria + 16 archaea mock genomes)
- ncbi_vs_gtdb_r89 conversion tables (data.gtdb.ecogenomic.org release89) for GTDBtk label conversion

## Outputs

- Per-MAG taxonomic classifications (strain/species/genus and higher ranks) filtered by the ass2ref threshold
- Benchmark accuracy table of sensitivity, precision, FPR, balanced accuracy, and F1-score per tool and rank level
- Mash ANI squared matrices rendered as heat maps for strain refinement

## How to apply

Bin contigs with MetaBAT2 (v2:2.15, --maxP 99 --minS 98 -m 1,500) to obtain MAGs, then run kMetaShot_classifier_NV.py against the ~22Gb HDF5 prokaryotic reference built from RefSeq CDSs/ncRNAs. Apply the ass2ref filter (ratio between MAG minimizers and reference minimizers of the assigned strain); the paper used 0.2 for all CAMI II datasets and the real mock community benchmark. For benchmarking, convert GTDBtk GTDB labels to NCBI taxonomy with ncbi_vs_gtdb_r89 conversion tables, then compare observed vs expected taxa to compute sensitivity, precision, FPR, balanced accuracy, and F1-score per rank. For strain-level calls on the mock community, refine assignments with Mash ANI (>=97%) against RefSeq genomes of observed strains. Allocate at least 32 Gb RAM for reference loading; with Docker pass --shm-size=22g.

## Related tools

- **kMetaShot** (primary alignment-free MAG taxonomy classifier based on k-mer/minimizer counting) — github.com/gdefazio/kMetaShot
- **GTDBtk** (marker-gene-based comparator tool; outputs converted from GTDB to NCBI taxonomy via ncbi_vs_gtdb_r89 tables)
- **CAMITAX** (comparator tool run through Nextflow (v21.04.0) for benchmarking classification accuracy)
- **MetaBAT2** (bins contigs into MAGs prior to classification (--maxP 99 --minS 98 -m 1,500))
- **Mash** (estimates pairwise ANI for strain-level assignment refinement (>=97% ANI))

## Examples

```
kMetaShot_classifier_NV.py -b bins/ -r kMetaShot_reference.h5 -p 10 -o output_dir -a 0.2
```

## Evaluation signals

- ass2ref values are between 0 and 1 and classifications below the chosen threshold (0.2 in the paper) are dropped as unreliable
- On CAMI II datasets, kMetaShot should attain top precision at species/genus levels and sensitivity medians such as 69.75 (IQR 3.86) at species level in GI-Illumina
- GTDBtk and CAMITAX should show no strain-level classification, whereas kMetaShot reports strain calls (HMP sensitivity ~82.0% at strain level)
- kMetaShot reference loading should peak at least 32 Gb of memory; Docker runs require --shm-size=22g to complete
- All compared tools' metrics (sensitivity, precision, FPR, BA, F1-score) are computed per rank level and aggregated across replicate samples (10 per GI/Air condition, 20 PA-PacBio samples)

## Limitations

- kMetaShot's peak memory and CPU correspond to loading the HDF5 reference in memory, requiring at least 32 Gb RAM
- High parallelism (-p processes) trades speed for RAM usage; the README warns 'high parallelism <==> high RAM usage'
- Docker containers fail without --shm-size=22g; CAMITAX resource usage must be measured via its Docker containers, not the Nextflow process
- GTDBtk/CAMITAX cannot classify at strain level, limiting rank-depth comparability; strain-level kMetaShot calls may require Mash ANI refinement for genomes lacking strain annotation

## Evidence

- [abstract] we present here kMetaShot, a taxonomy classifier based on k-mer/minimizer counting: "we present here kMetaShot, a taxonomy classifier based on k-mer/minimizer counting"
- [methods] the provided contigs were binned separately with MetaBAT2: "the provided contigs were binned separately with MetaBAT2 (v2:2.15) [35] (--maxP 99 --minS 98 -m 1,500) to obtain MAGs"
- [results] For kMetaShot, an ass2ref of 0.2 was set for all datasets.: "For kMetaShot, an ass2ref of 0.2 was set for all datasets."
- [discussion] the kMetaShot peaks in memory and CPU correspond to the reference loading in memory, requiring at least 32Gb: "the kMetaShot peaks in memory and CPU correspond to the reference loading in memory, requiring at least 32Gb."
- [readme] ass2ref is a ratio between the number of MAG minimizers: "ass2ref is a ratio between the number of MAG minimizers and the reference minimizers related to the assigned strain"
- [readme] It needs `--shm-size=22g` option to properly run as docker container.: "It needs `--shm-size=22g` option to properly run as docker container."
