---
name: pan-genome-index-construction
description: Use when you need to classify nanopore reads against one or many related
  reference genomes (e.g. one-genome vs multi-strain pan-genome FASTA) for targeted
  depletion, and must build MS and PML indexes plus a null index before running SPUMONI
  classification.
when_to_use_negative:
- Input is raw nanopore electrical signal — SPUMONI operates on base-called sequences,
  so a separate base caller must run upstream
- The task requires base-level alignments rather than classify/eject decisions — use
  minimap2 alignment instead
- There is substantial sequence similarity between depletion-target reads and non-target
  reads (e.g. conserved genes between species) — the reverse-sequence null index may
  be insufficient
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: SPUMONI
  role: Main tool whose 'spumoni build' command constructs the MS and PML pan-genome
    indexes
  repo: https://github.com/oma219/spumoni
- name: Big-BWT
  role: Prefix-free-parsing-based BWT construction used under the hood when building
    the index
  repo: https://github.com/alshai/Big-BWT
- name: r-index
  role: Provides the run-length-encoded BWT index structures consumed by the PML procedure
  repo: https://github.com/maxrossi91/r-index
- name: pfp-thresholds
  role: Computes the MONI threshold structures from prefix-free parsing used by index
    queries
  repo: https://github.com/maxrossi91/pfp-thresholds
- name: MONI
  role: Upstream batch MS-finding algorithm whose structures SPUMONI's index build
    reuses
  repo: https://github.com/maxrossi91/moni
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_007
  - task_005
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:4437ac458b47fbf068d284362fabed5eb004a45bf93e6db9fb57f46d3f34ad64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oma219/spumoni
  tools:
  - Big-BWT
  - Docker
  - MONI
  - PBSIM2
  - SPUMONI
  - Singularity
  - pfp-thresholds
  - r-index
  - sdsl-lite
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# pan-genome-index-construction

## Summary

Constructs SPUMONI pan-genome indexes containing both matching-statistics (MS) and pseudo-matching-length (PML) structures, plus a reverse-sequence null index, to support streaming nanopore read classification against large repetitive references.

## When to use

Use when you need to classify nanopore reads against one or many related reference genomes (e.g. one-genome vs multi-strain pan-genome FASTA) for targeted depletion, and must build MS and PML indexes plus a null index before running SPUMONI classification.

## When NOT to use

- Input is raw nanopore electrical signal — SPUMONI operates on base-called sequences, so a separate base caller must run upstream
- The task requires base-level alignments rather than classify/eject decisions — use minimap2 alignment instead
- There is substantial sequence similarity between depletion-target reads and non-target reads (e.g. conserved genes between species) — the reverse-sequence null index may be insufficient

## Inputs

- Reference FASTA (single genome, e.g. CHM13, or multi-FASTA pan-genome of many strains, e.g. CHM13 + Ashkenazi + GRCh38)
- Positive-index sequences with reverse complements added, for classification use
- Null index source: reverse (not reverse-complement) of the positive-index sequences

## Outputs

- SPUMONI index prefix containing the matching-statistics (MS) structures from -M
- SPUMONI index prefix containing the pseudo-matching-length (PML) structures from -P
- Run-length-encoded BWT and MONI threshold structures underlying PML computation

## How to apply

Install SPUMONI via the prebuilt Docker/Singularity image (oma219/spumoni:latest) or build from source (git clone, cmake .., make install), which pulls dependencies including Big-BWT, r-index, pfp-thresholds, sdsl-lite and libdivsufsort. Run 'spumoni build -r <ref.fa> -M -P -m -o <prefix>' on the reference FASTA to construct both the matching-statistics index (-M) and the pseudo-matching-length index (-P); the build produces the run-length-encoded BWT and MONI threshold structures that the PML procedure uses. For multi-genome pan-genome references (e.g. CHM13 + Ashkenazi + GRCh38), concatenate genomes into a single multi-FASTA before indexing. Optionally include target reference sequences AND their reverse complements in the positive index, and build a paired null index from the reverse (not reverse-complement) of those sequences to enable KS-stat hypothesis testing (thresholds: 0.25 for MS, 0.10 for PML). The pan-genome strategy maintains classification accuracy even when the exact target strain is absent (F1 99.7% on simulated 90%-accuracy ZymoMC reads), with an index roughly one-sixteenth the size of the equivalent minimap2 index.

## Related tools

- **SPUMONI** (Main tool whose 'spumoni build' command constructs the MS and PML pan-genome indexes) — https://github.com/oma219/spumoni
- **Big-BWT** (Prefix-free-parsing-based BWT construction used under the hood when building the index) — https://github.com/alshai/Big-BWT
- **r-index** (Provides the run-length-encoded BWT index structures consumed by the PML procedure) — https://github.com/maxrossi91/r-index
- **pfp-thresholds** (Computes the MONI threshold structures from prefix-free parsing used by index queries) — https://github.com/maxrossi91/pfp-thresholds
- **MONI** (Upstream batch MS-finding algorithm whose structures SPUMONI's index build reuses) — https://github.com/maxrossi91/moni

## Examples

```
spumoni build -r pangenome.fa -M -P -m -o pangenome_index
```

## Evaluation signals

- The built index prefix contains both MS (-M) and PML (-P) structures, since 'spumoni build -r <ref.fa> -M -P -m -o <prefix>' must construct both before classification can run in either mode
- Index size is consistent with reported scales (e.g. ~18 GB SPUMONI vs 21 GB minimap2 for the three-genome human index, and SPUMONI's many-strain index about one-sixteenth of minimap2's)
- On simulated E. coli reads where positive and null statistic distributions are known to differ, the positive and reverse-sequence null indexes yield distinguishable MS/PML distributions separable at the KS-stat thresholds (0.25 MS, 0.10 PML)
- Classification using the pan-genome index (target strain excluded) reaches F1 comparable to an index containing the exact target strain (~99.7% on simulated 90%-accuracy ZymoMC reads)
- Downstream 'spumoni run' against the built index produces a -c classification report without errors, confirming the index loads and contains the RLE BWT and MONI threshold structures

## Limitations

- The null index built from the reverse of positive-index sequences may be insufficient when depletion-target reads and non-target reads share substantial sequence similarity (e.g. conserved genes between species)
- A single KS-stat threshold is used for all experiments; the optimal threshold is a function of read accuracy and the reference used, and no per-experiment thresholding method is provided
- SPUMONI version number is not stated in the methods (only 'docker pull oma219/spumoni:latest'), so index reproducibility depends on tracking the image tag
- Throughput advantage over minimap2 is limited for human reads; the authors expect improvement only as the number of human genomes in the reference increases

## Evidence

- [methods] you could index a human genome and build both the matching statistic and pseudo-matching lengths index: "you could index a human genome and build both the matching statistic and pseudo-matching lengths index"
- [methods] Building an Index: "## Step 1: Building an Index"
- [intro] SPUMONI adds a null index together with a hypothesis testing framework to make principled ejection choices: "SPUMONI adds a “null index” together with a hypothesis testing framework to make principled ejection choices depending on whether the observed MS lengths are longer than what would be expected by"
- [methods] The easier way would be to use the Docker/Singularity images which have SPUMONI pre-installed and ready to use: "The easier way would be to use the Docker/Singularity images which have SPUMONI pre-installed and ready to use"
- [readme] SPUMONI is a software tool for performing rapid read classifications on sequencing reads using a read's matching statistics: "SPUMONI is a software tool for **performing rapid read classifications on sequencing reads using a read's matching statistics (or a related quantity called pseudo-matching lengths).**"
