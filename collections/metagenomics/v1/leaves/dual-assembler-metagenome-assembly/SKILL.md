---
name: dual-assembler-metagenome-assembly
description: Use when comparing whether a MAG taxonomic classifier's accuracy depends
  on the choice of meta-assembler, using independently reconstructed MAG sets from
  the same read dataset.
when_to_use_negative:
- When the goal is only to classify MAGs from a single assembly and assembler robustness
  is not in question — one assembler suffices.
- When the input is already binned MAGs from a mixed-provenance assembly, since binning
  from separate assembler outputs is central to the comparison.
- When evaluating reference-generator or ANI-clustering behaviour of kMetaShot rather
  than classification robustness across assembly pipelines.
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0637
tools:
- name: FastQC
  role: Quality check of paired-end mock community reads before trimming
- name: Sickle
  role: Trims reads with -q 32 -l 50 prior to assembly
- name: metaSPAdes
  role: First meta-assembler (v3.15.2, -k 35,57,79,99) producing one contig set
- name: MegaHIT
  role: Second meta-assembler (v1.2.9, -k 35,57,79,99) producing the comparison contig
    set
- name: MetaBAT2
  role: Bins contigs from each assembler separately (--maxP 99 --minS 98 -m 1500)
    into parallel MAG sets
- name: kMetaShot
  role: Classifies both MAG sets with ass2ref 0.2 against the HDF5 prokaryotic reference
  repo: https://github.com/gdefazio/kMetaShot
- name: Mash
  role: ANI comparison (>=97%) against RefSeq genomes for strain-level assignment
    refinement
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:fd6a26908f2118d617b0e3d0c510d6f891142ec065c477490b5d039a1b393b89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - FastQC
  - Mash
  - MegaHIT
  - MetaBAT2
  - Sickle
  - kMetaShot
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dual-assembler-metagenome-assembly

## Summary

Benchmark a downstream MAG taxonomy classifier (e.g. kMetaShot) against two independent meta-assemblies — metaSPAdes v3.15.2 and MegaHIT v1.2.9, both with k-mer list 35, 57, 79, 99 — of the same mock-community reads, to test whether classification reliability is assembler-independent.

## When to use

Use when comparing whether a MAG taxonomic classifier's accuracy depends on the choice of meta-assembler, using independently reconstructed MAG sets from the same read dataset.

## When NOT to use

- When the goal is only to classify MAGs from a single assembly and assembler robustness is not in question — one assembler suffices.
- When the input is already binned MAGs from a mixed-provenance assembly, since binning from separate assembler outputs is central to the comparison.
- When evaluating reference-generator or ANI-clustering behaviour of kMetaShot rather than classification robustness across assembly pipelines.

## Inputs

- Paired-end shotgun metagenomic reads of a mock community (e.g. SRR606249 FASTQ files)
- metaSPAdes-assembled contigs and MegaHIT-assembled contigs (same k-mer list 35,57,79,99)
- MetaBAT2 MAG sets binned separately from each assembler's contigs
- kMetaShot HDF5 prokaryotic reference (kMetaShot_bacteria_archaea_*.h5)

## Outputs

- Two MAG sets (metaSPAdes-derived and MegaHIT-derived) binned with MetaBAT2
- kMetaShot classifications for both MAG sets, before and after Mash-based strain refinement
- Assembler_comparison_metrics.csv with sensitivity, precision, FDR, balanced accuracy and F1-score for both assemblers

## How to apply

Quality-check the paired-end mock reads (Shakya et al., SRA SRR606249) with FastQC and trim with Sickle (-q 32 -l 50). Meta-assemble the trimmed reads independently with metaSPAdes v3.15.2 and MegaHIT v1.2.9, each with k 35, 57, 79, 99. Bin contigs from each assembler separately with MetaBAT2 v2:2.15 (--maxP 99 --minS 98 -m 1500) to yield two parallel MAG sets. Classify both MAG sets with kMetaShot against the HDF5 prokaryotic reference using the prevalence-based classifier with ass2ref set to 0.2; resolve strain-level assignments lacking reference strain annotation by ANI comparison with Mash (>=97%). Compute sensitivity, precision, FDR, balanced accuracy and F1-score for both assemblers side by side and confirm performance is comparable across the two MAG sets.

## Related tools

- **FastQC** (Quality check of paired-end mock community reads before trimming)
- **Sickle** (Trims reads with -q 32 -l 50 prior to assembly)
- **metaSPAdes** (First meta-assembler (v3.15.2, -k 35,57,79,99) producing one contig set)
- **MegaHIT** (Second meta-assembler (v1.2.9, -k 35,57,79,99) producing the comparison contig set)
- **MetaBAT2** (Bins contigs from each assembler separately (--maxP 99 --minS 98 -m 1500) into parallel MAG sets)
- **kMetaShot** (Classifies both MAG sets with ass2ref 0.2 against the HDF5 prokaryotic reference) — https://github.com/gdefazio/kMetaShot
- **Mash** (ANI comparison (>=97%) against RefSeq genomes for strain-level assignment refinement)

## Examples

```
kMetaShot_classifier_NV.py -b bins_metabat2/ -r kMetaShot_bacteria_archaea_2025-05-22.h5 -p 10 -o output_dir -a 0.2
```

## Evaluation signals

- The metrics table reports all five metrics (sensitivity, precision, FDR, balanced accuracy, F1-score) for both assemblers.
- kMetaShot assigns the highest number of MAGs with the lowest number of false positives at species and genus levels on BOTH the metaSPAdes- and MegaHIT-derived MAG sets.
- The two assembler-derived MAG sets are binned independently with identical MetaBAT2 parameters (--maxP 99 --minS 98 -m 1500).
- Strain-level assignments without reference strain annotation are resolved by Mash ANI >= 97% against RefSeq genomes in both arms of the comparison.
- Concluding check: observed differences between assemblers are small, supporting assembler-independent classification as claimed in the paper.

## Limitations

- The comparison was validated on a single real mock community dataset (Shakya et al., SRR606249); conclusions may not transfer to complex environmental communities.
- kMetaShot classification uses a prevalence-based approach with the ass2ref filter (0.2 in this benchmark); other thresholds could change results.
- kMetaShot's reference loading in memory requires at least 32Gb and the HDF5 reference needs about 22Gb of storage.
- As a Docker container, kMetaShot needs the --shm-size=22g option to run properly.

## Evidence

- [methods] PE reads were meta-assembled with metaSpades (v3.15.2) [44] and MegaHIT (v1.2.9) [45] (for both -k 35, 57, 79, 99): "PE reads were meta-assembled with metaSpades (v3.15.2) [44] and MegaHIT (v1.2.9) [45] (for both -k 35, 57, 79, 99)"
- [methods] MetaBAT2 binning parameters: "the provided contigs were binned separately with MetaBAT2 (v2:2.15) [35] (--maxP 99 --minS 98 -m 1,500) to obtain MAGs"
- [results] ass2ref filter: "we implemented ass2ref (the ratio among the observed and expected taxa minimizer), as a measure of the reliability of the kMetaShot classifications."
- [readme] kMetaShot classifier invocation: "ass2ref is a ratio between the number of MAG minimizers and the reference minimizers related to the assigned strain"
- [results] kMetaShot is assembler independent: [evidence span withheld — anchor status: not_found]
