---
name: average-nucleotide-identity-clustering
description: Use when a quality-controlled collection of prokaryotic genome FASTAs
  (MAGs plus isolates) must be dereplicated into species-level bins at 95% ANI, or
  when reproducing a published species count such as the VMGC's 786 SGBs.
when_to_use_negative:
- Viral or fungal genomes — the VMGC dereplicates viruses into 4,263 vOTUs and processes
  fungi through separate pipelines (e.g. virusClstr.sh), not the prokaryotic 95% ANI
  species clustering.
- Strain- or subspecies-level resolution is required — the 95% ANI / 30% coverage
  cutoff deliberately collapses strains into single species bins.
- Genomes have not passed the >=50% completeness and <5% contamination quality screen,
  so low-quality or contaminated MAGs would distort ANI-based clustering.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dRep
  role: Runs the dereplicate workflow (v3.4.0) that pre-groups genomes at 90% primary
    ANI and forms the 95% ANI species clusters from the 19,542 VMGC prokaryotic FASTAs
- name: fastANI
  role: Secondary all-versus-all ANI engine selected via dRep --S_algorithm fastANI;
    computes the genome-pair ANIs and alignment coverage behind the -sa 0.95 / -nc
    0.3 cutoff
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:e62a452fe0fd5e721a9a6e308f1c9a26d155693280464146b607a9847b2f4c9a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - dRep
  - fastANI
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# average-nucleotide-identity-clustering

## Summary

Species-level dereplication of a quality-controlled prokaryotic genome collection by average nucleotide identity (ANI): the 19,542 VMGC prokaryotic genomes (18,570 MAGs plus 972 isolate/reference genomes) are clustered with dRep/fastANI at a 95% ANI threshold to recover 786 species-level genome bins (SGBs) and a nonredundant representative genome set.

## When to use

Use when a quality-controlled collection of prokaryotic genome FASTAs (MAGs plus isolates) must be dereplicated into species-level bins at 95% ANI, or when reproducing a published species count such as the VMGC's 786 SGBs.

## When NOT to use

- Viral or fungal genomes — the VMGC dereplicates viruses into 4,263 vOTUs and processes fungi through separate pipelines (e.g. virusClstr.sh), not the prokaryotic 95% ANI species clustering.
- Strain- or subspecies-level resolution is required — the 95% ANI / 30% coverage cutoff deliberately collapses strains into single species bins.
- Genomes have not passed the >=50% completeness and <5% contamination quality screen, so low-quality or contaminated MAGs would distort ANI-based clustering.

## Inputs

- Prokaryotic genome FASTA files: VMGC_prokaryote_MAG.tar.gz (n=19,542; 18,570 quality-controlled MAGs + 972 isolate/reference genomes) from https://zenodo.org/records/10457006
- Per-genome annotations with Source/Quality/Clustering columns (VMGC_prokaryote_MAG.info) confirming which genomes enter dereplication and providing cross-check data

## Outputs

- sgb_clustering.tsv: genome-to-cluster assignments defining 786 species-level genome bins (SGBs)
- dRep secondary clustering outputs (e.g. secondary_cluster.csv and fastANI ANI tables) from which SGB assignments are parsed
- Nonredundant representative genome FASTA set (n=786), equivalent to VMGC_prokaryote_SGB.tar.gz (440 MB)

## How to apply

Unpack VMGC_prokaryote_MAG.tar.gz from Zenodo record 10457006, giving 19,542 FASTAs (18,570 MAGs that passed the >=50% completeness and <5% contamination screen plus 972 isolate/RefSeq genomes), and run dRep v3.4.0 in dereplicate mode. Set -pa 0.9 so genomes are first pre-grouped at 90% ANI, pruning the expensive all-versus-all stage, then compute exact secondary ANI with --S_algorithm fastANI. Impose the species cutoff with -sa 0.95 and require -nc 0.3 so a genome pair joins a bin only when the alignment covers at least 30% of the shorter genome, guarding against short shared regions driving false merges of fragmented MAGs. Parse dRep's secondary clustering table into a genome-to-SGB mapping (sgb_clustering.tsv) and, optionally, keep one representative FASTA per bin to mirror the 440 MB VMGC_prokaryote_SGB.tar.gz (n=786). Judge the run correct only when the cluster count equals 786 and membership matches the Clustering annotations in VMGC_prokaryote_MAG.info, Supplementary Table 3 and Fig. 2a.

## Related tools

- **dRep** (Runs the dereplicate workflow (v3.4.0) that pre-groups genomes at 90% primary ANI and forms the 95% ANI species clusters from the 19,542 VMGC prokaryotic FASTAs)
- **fastANI** (Secondary all-versus-all ANI engine selected via dRep --S_algorithm fastANI; computes the genome-pair ANIs and alignment coverage behind the -sa 0.95 / -nc 0.3 cutoff)

## Examples

```
dRep dereplicate sgb_out -g genomes/*.fasta -pa 0.9 -sa 0.95 -nc 0.3 --S_algorithm fastANI
```

## Evaluation signals

- The number of parsed clusters equals exactly 786 species-level genome bins for the 19,542-genome input set.
- Every input genome appears in exactly one SGB row of sgb_clustering.tsv (complete, non-overlapping assignment).
- Cluster membership agrees with the Clustering column of VMGC_prokaryote_MAG.info, Supplementary Table 3 and Fig. 2a.
- The dereplicated representative set contains 786 FASTAs, matching VMGC_prokaryote_SGB.tar.gz (n=786, 440 MB).
- Intra-cluster fastANI pairs satisfy ANI >=0.95 with >=30% coverage of the shorter genome, consistent with -sa 0.95 and -nc 0.3.

## Limitations

- The article notes CheckM2 completeness might be underestimated for some taxa (e.g. A. indicium and A. nucleatus), so upstream quality filtering can shift which genomes enter clustering.
- The 95% ANI plus 30% coverage cutoff merges strains into species bins, so within-species (strain-level) diversity is not represented among the 786 SGBs.
- Fragmented MAGs sharing less than 30% of the shorter genome may split from their species even at high ANI, and results are sensitive to dRep version and parameters (-pa, -sa, -nc, S_algorithm).

## Evidence

- [abstract] 786 SGBs defined at 95% nucleotide similarity: [evidence span withheld — anchor status: not_found]
- [abstract] Quality screen that defines the clustering input set: "yielded 18,570 prokaryotic genomes with ≥50% completeness and <5% contamination"
- [readme] Full prokaryotic genome input file: "Full prokaryotic genomes (n=19,542)  | 8.1 GB | VMGC_prokaryote_MAG.tar.gz"
- [readme] Expected nonredundant output count: "Nonredundant prokaryotic genomes (n=786) | 440 MB | VMGC_prokaryote_SGB.tar.gz"
- [other] dRep/fastANI parameters for the species clustering run: [evidence span withheld — anchor status: not_found]
