---
name: sintax-bootstrap-taxonomy-classification
description: Use when classifying full-length or amplicon 16S rRNA FASTA sequences
  against a reference database (e.g. Greengenes2 2024.09) with a fast k-mer classifier
  and needing per-rank bootstrap confidence assignments as a benchmark baseline.
when_to_use_negative:
- Input is a feature table or already-classified taxonomy profiles rather than raw
  FASTA amplicon sequences
- 'Maximum species-level accuracy is required: SINTAX''s 0.8 bootstrap cutoff returns
  many unclassified calls and it underperforms DADA2, QIIME 2, and DeepTaxa on the
  same Greengenes2 test set'
- Evaluating short V3-V4/V4 amplicons where a model trained specifically on in-silico
  amplicons (e.g. DeepTaxa V3-V4 checkpoint, 87.55%) may be more appropriate
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0084
tools:
- name: SINTAX
  role: k-mer-based taxonomic classifier applied with a 0.8 bootstrap-confidence cutoff
    to produce per-rank assignments and the 73.13% species-accuracy baseline
- name: USEARCH
  role: executable (v11.0.667) in which SINTAX runs against the Greengenes2 2024.09
    FASTA formatted to SINTAX requirements
- name: DeepTaxa
  role: hybrid CNN-BERT framework whose 92.96% species accuracy on the same test set
    provides the comparison target for the SINTAX baseline
  repo: https://github.com/systems-genomics-lab/deeptaxa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:0d2dec353636d0289579b16cac3d5127718cc93ba52c6c8d1974ce90da02ba3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DeepTaxa
  - Kraken 2
  - QIIME 2
  - SINTAX
  - USEARCH
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# sintax-bootstrap-taxonomy-classification

## Summary

Species-level taxonomic classification of 16S rRNA amplicon sequences using SINTAX, the k-mer-based naive-Bayes classifier in USEARCH, with bootstrap confidence support. In the DeepTaxa benchmark it serves as one of four established baselines on the Greengenes2 2024.09 held-out test set, reaching 73.13% species-level top-1 accuracy.

## When to use

Use when classifying full-length or amplicon 16S rRNA FASTA sequences against a reference database (e.g. Greengenes2 2024.09) with a fast k-mer classifier and needing per-rank bootstrap confidence assignments as a benchmark baseline.

## When NOT to use

- Input is a feature table or already-classified taxonomy profiles rather than raw FASTA amplicon sequences
- Maximum species-level accuracy is required: SINTAX's 0.8 bootstrap cutoff returns many unclassified calls and it underperforms DADA2, QIIME 2, and DeepTaxa on the same Greengenes2 test set
- Evaluating short V3-V4/V4 amplicons where a model trained specifically on in-silico amplicons (e.g. DeepTaxa V3-V4 checkpoint, 87.55%) may be more appropriate

## Inputs

- Held-out test set of 69,335 full-length Greengenes2 2024.09 16S rRNA sequences (FASTA)
- Greengenes2 2024.09 training/reference FASTA (277,336 training sequences) reformatted to SINTAX header conventions

## Outputs

- Per-sequence taxonomy assignments with bootstrap confidence scores at each rank (domain through species)
- Per-rank top-1 accuracy on the shared 69,335-sequence test set (species-level: 73.13%)
- Wall-clock classification time recorded for benchmark comparison

## How to apply

Format the Greengenes2 2024.09 reference FASTA (346,671 curated 16S sequences) to SINTAX requirements so taxonomy is embedded in the FASTA headers in the ';tax=d:p,c:o,...' convention. Run SINTAX from USEARCH v11.0.667 against this database to classify the shared 69,335-sequence held-out test set. Apply the recommended 0.8 bootstrap-confidence cutoff: assignments whose species-rank bootstrap support falls below 0.8 are returned as unclassified, which is the documented reason SINTAX's species accuracy (73.13%) trails DADA2 (90.05%) and QIIME 2 (85.01%). Compute per-rank top-1 accuracy on the shared test set, treating unclassified calls as incorrect, and record wall-clock classification time for benchmarking against DeepTaxa (92.96%).

## Related tools

- **SINTAX** (k-mer-based taxonomic classifier applied with a 0.8 bootstrap-confidence cutoff to produce per-rank assignments and the 73.13% species-accuracy baseline)
- **USEARCH** (executable (v11.0.667) in which SINTAX runs against the Greengenes2 2024.09 FASTA formatted to SINTAX requirements)
- **DeepTaxa** (hybrid CNN-BERT framework whose 92.96% species accuracy on the same test set provides the comparison target for the SINTAX baseline) — https://github.com/systems-genomics-lab/deeptaxa

## Examples

```
usearch -sintax gg_2024_09_testing.fna -db gg_2024_09_sintax.fna -strand both -sintax_cutoff 0.8 -tabbedout sintax_species.tsv
```

## Evaluation signals

- Species-level top-1 accuracy on the shared 69,335-sequence Greengenes2 test set falls within a small tolerance of the reported 73.13%
- SINTAX ranks lowest among the four benchmark classifiers (DADA2 90.05%, QIIME 2 85.01%, Kraken 2 28.05%) except that Kraken 2 is lowest overall — SINTAX should sit between QIIME 2 and Kraken 2
- A substantial fraction of sequences are returned as unclassified at the species rank under the 0.8 bootstrap cutoff, consistent with the documented cause of the reduced accuracy
- All tools (including SINTAX) are evaluated on the same held-out test set derived from the same Greengenes2 2024.09 reference FASTA, ensuring a like-for-like comparison
- Per-rank accuracy is tabulated (benchmark_species_accuracy.csv) for domain through species, with wall-clock time recorded

## Limitations

- Under the recommended 0.8 bootstrap-confidence cutoff, many species-level assignments fall below the threshold and are returned as unclassified, lowering species accuracy to 73.13%
- As a k-mer-based classifier it shows larger deficits than alignment-based methods against deep learning classifiers on closely related species and large reference databases
- Accuracy is reference-database specific: results were only established on Greengenes2 2024.09, not on SILVA or RDP

## Evidence

- [results] SINTAX bootstrap cutoff lowers accuracy: "under its recommended 0.8 bootstrap-confidence cutoff, many species-level assignments fall below the threshold and are returned as unclassified, lowering its species accuracy to 73.13%"
- [methods] Benchmark design: "we conducted a benchmarking study comparing its performance against widely used taxonomic classification tools, including DADA2 (Callahan et al. 2016), QIIME 2, Kraken2, and SINTAX"
- [abstract] Overall comparison finding: "DeepTaxa exceeds DADA2 (90.05%) and QIIME 2 (85.01%) at the species rank on the same held-out test set, with larger gains over the k-mer-based classifiers SINTAX and Kraken 2"
- [readme] Test set size: [evidence span withheld — anchor status: external_doc]
