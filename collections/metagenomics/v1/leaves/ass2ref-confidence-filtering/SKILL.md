---
name: ass2ref-confidence-filtering
description: 'Use when classifying MAGs with kMetaShot and you need to discard unreliable
  strain- or species-level assignments: apply the -a/ass2ref filter to the classifier
  output so only classifications whose minimizer support meets the ratio threshold
  are retained.'
when_to_use_negative:
- You are using a marker-gene/alignment classifier such as GTDBtk or CAMITAX, which
  rely on marker gene identification and sequence alignments rather than minimizer
  ratios
- You need classifications of MAGs against a custom reference not represented in the
  kMetaShot prokaryotic RefSeq reference, since ass2ref is defined against the assigned
  strain's reference minimizers
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: kMetaShot
  role: k-mer/minimizer-based classifier whose ass2ref parameter (ratio among observed
    and expected taxa minimizers) filters unreliable classifications
  repo: https://github.com/gdefazio/kMetaShot
- name: conda
  role: Installs kMetaShot 2.0 from the bioconda channel with Python 3.9
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:d0687f7d3f54e16b925895e1a494562c297d2c4b4071c6c91d56c3d92bed6656
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - Python
  - conda
  - kMetaShot
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ass2ref-confidence-filtering

## Summary

Apply kMetaShot's ass2ref ratio — the ratio between the observed MAG minimizers and the reference minimizers of the assigned strain — as a reliability measure to drop out unreliable strain/species classifications, typically with a threshold of 0.2.

## When to use

Use when classifying MAGs with kMetaShot and you need to discard unreliable strain- or species-level assignments: apply the -a/ass2ref filter to the classifier output so only classifications whose minimizer support meets the ratio threshold are retained.

## When NOT to use

- You are using a marker-gene/alignment classifier such as GTDBtk or CAMITAX, which rely on marker gene identification and sequence alignments rather than minimizer ratios
- You need classifications of MAGs against a custom reference not represented in the kMetaShot prokaryotic RefSeq reference, since ass2ref is defined against the assigned strain's reference minimizers

## Inputs

- MAG FASTA bins (directory of .fa/.fasta/.fna/.gz files) or a multi-fasta with one header per MAG
- HDF5 kMetaShot reference (e.g. kMetaShot_bacteria_archaea_2025-05-22.h5, ~22 Gb)
- ass2ref threshold (float, 0–1; 0.2 in the article's benchmarks)

## Outputs

- kmetashot_classifications.csv with one line per classified MAG and 11 fields (bin name, ass2ref, taxid, taxonomy species→kingdom, assigned microorganism name)
- Set of retained classifications whose ass2ref ≥ threshold; low-ass2ref strain/species calls dropped

## How to apply

Run kMetaShot_classifier_NV.py with the -a/--ass2ref flag set to a threshold between 0 and 1 (the article used 0.2 for all CAMI II datasets and the benchmark's custom filter was also set to 0.2); the classifier shreds each MAG into q-minimizers, queries the HDF5 storage matrix, and computes ass2ref as the ratio of observed MAG minimizers to the reference minimizers of the assigned strain. Classifications falling below the threshold are dropped as unreliable, while among survivors kMetaShot assigns taxonomy via a prevalence-based approach (most abundant genus, then species, then the strain maximizing ass2ref). Each output CSV line reports the achieved ass2ref value alongside taxid and taxonomy, so the filter both removes weak calls and quantifies the confidence of retained ones.

## Related tools

- **kMetaShot** (k-mer/minimizer-based classifier whose ass2ref parameter (ratio among observed and expected taxa minimizers) filters unreliable classifications) — https://github.com/gdefazio/kMetaShot
- **conda** (Installs kMetaShot 2.0 from the bioconda channel with Python 3.9)

## Examples

```
kMetaShot_classifier_NV.py -b bins/ -r kMetaShot_reference/kMetaShot_bacteria_archaea.h5 -p 10 -o output_dir -a 0.2
```

## Evaluation signals

- kmetashot_classifications.csv exists and every line contains the 11 declared fields including bin name, ass2ref, taxid, taxonomy, and assigned microorganism name
- Every reported ass2ref value lies in the range 0–1
- No retained classification has an ass2ref below the chosen threshold (e.g. 0.2)
- Classification rates at strain/species/genus levels are consistent with the filtered benchmark (HMP: 770/830/872 correct assignments of 939 genomes; sensitivity ~82.0%/89.2%/92.9%)
- Installation verified with kMetaShot_test.py -r /path/to/kMetaShot_reference.h5 before filtering

## Limitations

- Reference loading requires at least 32 Gb RAM and the HDF5 reference needs ~22 Gb of storage; high parallelism increases RAM usage
- ass2ref is defined only relative to the assigned strain's reference minimizers, so it cannot flag false assignments to an incorrect strain with sufficient minimizer overlap
- An ass2ref value of 0 (default) applies no filtering, retaining potentially unreliable calls

## Evidence

- [readme] ass2ref is a ratio between the number of MAG minimizers and the reference minimizers related to the assigned strain: "ass2ref is a ratio between the number of MAG minimizers and the reference minimizers related to the assigned strain"
- [results] we implemented ass2ref (the ratio among the observed and expected taxa minimizer), as a measure of the reliability of the kMetaShot classifications.: "we implemented ass2ref (the ratio among the observed and expected taxa minimizer), as a measure of the reliability of the kMetaShot classifications."
- [methods] the ass2ref parameter (i.e. ranging between 0 and 1) was implemented as an optional filter to drop out unreliable classifications: "the ass2ref parameter (i.e. ranging between 0 and 1) was implemented as an optional filter to drop out unreliable classifications"
- [results] For kMetaShot, an ass2ref of 0.2 was set for all datasets.: "For kMetaShot, an ass2ref of 0.2 was set for all datasets."
