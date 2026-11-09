---
name: classification-metrics-evaluation
description: Use when a taxonomic classifier (e.g., kMetaShot vs GTDBtk vs CAMITAX)
  has been run on genomes with known expected labels and you need rank-level (strain/species/genus)
  performance metrics.
when_to_use_negative:
- The classifier output taxonomy differs from the expected-label taxonomy and no conversion
  table (e.g., ncbi_vs_gtdb_r89) is available — metrics would be biased by nomenclature
  differences rather than classifier error.
- You need per-read or per-sample abundance accuracy (e.g., CAMI II mock community
  profiling) rather than set-based per-genome classification agreement.
- Expected ground-truth labels for the genomes are not available, making the CO∩CE
  intersection undefined.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3174
tools:
- name: kMetaShot
  role: Classifier under evaluation; its classifications (CO) at strain/species/genus
    levels are scored against expected HMP labels
  repo: https://github.com/gdefazio/kMetaShot
- name: GTDBtk
  role: Comparison classifier (v1.0.2); genus/species-level outputs converted from
    GTDB to NCBI taxonomy before metric computation
  repo: https://data.gtdb.ecogenomic.org/releases/release89/89.0/
- name: CAMITAX
  role: Comparison classifier (v0.7.0) benchmarked with the same set-based metrics;
    provides no strain-level classification
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:8b21c3254efbfe8577e7b21bbda06e1dfb46c0ded042b0556db5d271c45302f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - CAMITAX
  - GTDBtk
  - conda
  - kMetaShot
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# classification-metrics-evaluation

## Summary

A set-based evaluation procedure for taxonomic classification benchmarks that computes sensitivity, precision, false positive rate, balanced accuracy, and F1-score at strain, species, and genus levels by comparing observed classifications (CO) against expected labels (CE) on a fixed genome set.

## When to use

Use when a taxonomic classifier (e.g., kMetaShot vs GTDBtk vs CAMITAX) has been run on genomes with known expected labels and you need rank-level (strain/species/genus) performance metrics.

## When NOT to use

- The classifier output taxonomy differs from the expected-label taxonomy and no conversion table (e.g., ncbi_vs_gtdb_r89) is available — metrics would be biased by nomenclature differences rather than classifier error.
- You need per-read or per-sample abundance accuracy (e.g., CAMI II mock community profiling) rather than set-based per-genome classification agreement.
- Expected ground-truth labels for the genomes are not available, making the CO∩CE intersection undefined.

## Inputs

- Expected taxonomic labels per genome (e.g., NCBI-based labels from HMP flat files, 939 bacterial and archaeal HMP genomes spanning 934 strains, 556 species, 190 genera)
- Classifier output tables per genome per rank (e.g., kMetaShot results, GTDBtk v1.0.2 results converted via ncbi_vs_gtdb_r89 tables, CAMITAX results)
- Count of genomes the classifier attempted (CT) versus total genome set

## Outputs

- Per-rank metric table: sensitivity, precision, FPR, balanced accuracy, F1-score at strain, species, and genus levels
- Correct assignment counts per rank (e.g., Tp values such as 770/830/872 out of 939 genomes)

## How to apply

Extract expected taxonomic labels (CE) for each genome from the source annotations (e.g., HMP flat files from the NCBI Human Microbiome FTP site) and collect the classifier outputs (CO) and counts of classified genomes (CT). For each taxonomic rank, compute Tp = CO∩CE, Fp = CO−(CO∩CE), Fn = CE−(CO∩CE), and Tn = CT−(CO∩CE), then derive sensitivity, precision, FPR, balanced accuracy, and F1-score. Compute metrics separately at strain, species, and genus levels, noting that tools like GTDBtk and CAMITAX provide no strain-level classification and their outputs may require taxonomy conversion (e.g., GTDB to NCBI using ncbi_vs_gtdb_r89 conversion tables) before comparison. As a sanity anchor, the kMetaShot benchmark on 939 HMP genomes reported 770/830/872 correct assignments at strain/species/genus (sensitivity ~82.0%/89.2%/92.9%, BA 90.8–95.4%, FPR 0.2–1.9%).

## Related tools

- **kMetaShot** (Classifier under evaluation; its classifications (CO) at strain/species/genus levels are scored against expected HMP labels) — https://github.com/gdefazio/kMetaShot
- **GTDBtk** (Comparison classifier (v1.0.2); genus/species-level outputs converted from GTDB to NCBI taxonomy before metric computation) — https://data.gtdb.ecogenomic.org/releases/release89/89.0/
- **CAMITAX** (Comparison classifier (v0.7.0) benchmarked with the same set-based metrics; provides no strain-level classification)

## Examples

```
python compute_metrics.py --expected hmp_expected_labels.tsv --observed kmetashot_output/ --ranks strain,species,genus --total 939
```

## Evaluation signals

- Sensitivity falls within the reported ranges for the reference benchmark: ~82.0% (strain), ~89.2% (species), ~92.9% (genus) on the 939 HMP genomes.
- FPR values fall within 0.2% (strains and species) to 1.9% (genus).
- Balanced accuracy lies between 90.8% and 95.4% across ranks.
- Correct assignment counts match reported Tp values (770, 830, 872 at strain, species, genus levels) with 926 of 939 genomes classified and only 11 strains and 2 species left unclassified.
- Strain-level metrics are reported only for classifiers that support strain-level output (GTDBtk and CAMITAX provide none).

## Limitations

- GTDBtk and CAMITAX provide no strain-level classification, so strain-rank comparison is only possible with tools like kMetaShot.
- Comparators' outputs require taxonomy-space conversion (GTDB to NCBI via ncbi_vs_gtdb_r89 release 89 tables); errors in conversion propagate into the metrics.
- Set-based definitions treat each rank independently and depend on complete expected-label annotation; unclassified genomes reduce sensitivity even if the classifier is conservative rather than wrong.

## Evidence

- [full_text] Compute sensitivity, precision, FPR, balanced accuracy, and F1-score at strain, species, and genus levels using the paper's set-based definitions: [evidence span withheld — anchor status: not_found]
- [results] It correctly assigned 770, 830, and 872 genomes at strain, species, and genus levels, respectively: "It correctly assigned 770, 830, and 872 genomes at strain, species, and genus levels, respectively"
- [results] we observed values ranging from 0.2% (strains and species) to 1.9% (genus).: "we observed values ranging from 0.2% (strains and species) to 1.9% (genus)."
- [results] The 939 (937 bacteria and 2 archaea) HMP genomes span 190 genera, 556 species, and 934 strains: "The 939 (937 bacteria and 2 archaea) HMP genomes span 190 genera, 556 species, and 934 strains"
- [methods] using the ncbi_vs_gtdb_r89 conversion tables (available at https://data.gtdb.ecogenomic.org/releases/release89/89.0/): "using the ncbi_vs_gtdb_r89 conversion tables (available at https://data.gtdb.ecogenomic.org/releases/release89/89.0/)"
