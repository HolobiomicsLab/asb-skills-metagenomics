---
name: prokaryotic-genome-dereplication
description: Use when a large set of assembled prokaryotic genomes (MAGs plus isolates,
  FASTA format) must be collapsed into nonredundant species-level bins at ~95% nucleotide
  similarity before taxonomy assignment or downstream cataloguing.
when_to_use_negative:
- Input genomes are viral or eukaryotic (VMGC clusters viruses into 4,263 vOTUs and
  fungi separately; the 95% ANI dRep SGB workflow applies to prokaryotic genomes only)
- Genomes have not passed quality screening (VMGC applies the ≥50% completeness /
  <5% contamination MIMAG-revised filter before dereplication)
- The goal is gene-level clustering of protein-coding genes at 50/90/95% amino acid
  identity (VMGC-50/90/95 catalogues), not genome-level species dereplication
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: dRep
  role: Genome dereplication/clustering engine; run v3.4.0 in dereplicate mode with
    -pa 0.9, -sa 0.95, -nc 0.3 to form the 786 SGBs
- name: fastANI
  role: Secondary all-versus-all ANI algorithm invoked by dRep via --S_algorithm fastANI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:90cb75c3c4f18e7acc991bcf30a97ffc78c3dc986847ab04499ece9ea658a2b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - dRep
  - fastANI
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# prokaryotic-genome-dereplication

## Summary

Dereplicates a large collection of quality-controlled prokaryotic MAGs plus isolate/reference genomes into species-level genome bins (SGBs) using dRep with a 95% ANI species threshold, as done to collapse the 19,542 VMGC prokaryotic genomes into 786 SGBs for the vaginal microbiome reference catalogue.

## When to use

Use when a large set of assembled prokaryotic genomes (MAGs plus isolates, FASTA format) must be collapsed into nonredundant species-level bins at ~95% nucleotide similarity before taxonomy assignment or downstream cataloguing.

## When NOT to use

- Input genomes are viral or eukaryotic (VMGC clusters viruses into 4,263 vOTUs and fungi separately; the 95% ANI dRep SGB workflow applies to prokaryotic genomes only)
- Genomes have not passed quality screening (VMGC applies the ≥50% completeness / <5% contamination MIMAG-revised filter before dereplication)
- The goal is gene-level clustering of protein-coding genes at 50/90/95% amino acid identity (VMGC-50/90/95 catalogues), not genome-level species dereplication

## Inputs

- Quality-controlled prokaryotic genome FASTA files (MAGs ≥50% completeness, <5% contamination, e.g. 18,570 VMGC MAGs)
- Isolate/reference prokaryotic genomes (e.g. 972 NCBI RefSeq genomes included in VMGC)
- Zenodo archive VMGC_prokaryote_MAG.tar.gz (n=19,542 full prokaryotic genomes)

## Outputs

- Species-level genome bin (SGB) assignments for every input genome (e.g. sgb_clustering.tsv; 786 SGBs for VMGC)
- Nonredundant representative genome FASTAs per SGB (VMGC_prokaryote_SGB.tar.gz, n=786)
- Cluster annotation table with taxonomy/quality/clustering info (VMGC_prokaryote_SGB.info)

## How to apply

Load the quality-filtered genome FASTAs (in VMGC: 18,570 MAGs passing ≥50% completeness and <5% contamination, plus 972 isolate/reference genomes from NCBI RefSeq; distributed as VMGC_prokaryote_MAG.tar.gz). Run dRep v3.4.0 in dereplicate mode with -pa 0.9 for primary ANI prefiltering, -sa 0.95 as the species-level ANI threshold, -nc 0.3 requiring ≥30% coverage of the shorter genome, and --S_algorithm fastANI for the secondary all-versus-all ANI computation. Parse the dRep secondary-cluster output to map each genome to its species-level genome bin and export the assignments (e.g. as sgb_clustering.tsv). The expected output for the VMGC input is exactly 786 SGBs, which also feed downstream artefacts such as the nonredundant VMGC_prokaryote_SGB.tar.gz genome set, the SGB.info/sgb.info annotations, and the custom Kraken2/Bracken database built from the 786 SGBs.

## Related tools

- **dRep** (Genome dereplication/clustering engine; run v3.4.0 in dereplicate mode with -pa 0.9, -sa 0.95, -nc 0.3 to form the 786 SGBs)
- **fastANI** (Secondary all-versus-all ANI algorithm invoked by dRep via --S_algorithm fastANI)

## Examples

```
dRep dereplicate sgb_out -g genomes/*.fasta -pa 0.9 -sa 0.95 -nc 0.3 --S_algorithm fastANI
```

## Evaluation signals

- Number of species-level clusters equals the reported 786 SGBs for the 19,542-genome VMGC prokaryotic input
- Every input genome is assigned to exactly one SGB (no unclustered or multiply-assigned genomes in sgb_clustering.tsv)
- Cluster membership cross-checks against Supplementary Table 3 and the clustering shown in Fig. 2a
- Representative SGB genome set size (VMGC_prokaryote_SGB.tar.gz) equals 786 and matches the SGB annotation file row count
- Custom Kraken2/Bracken database built from the SGBs (kraken_MAGdb.sh) reproduces the taxonomy derived from GTDB-tk annotations of the 786 bins

## Limitations

- Dereplication collapses to a 95% ANI species definition; uncultured species (208 of 786 SGBs, 26.5% of VMGC prokaryotic species) are grouped by sequence similarity alone without cultivated type-strain confirmation
- Potential contamination from experimental operations or kits in the public vaginal metagenomic datasets feeding the MAGs propagates into the SGBs
- The VMGC itself is incomplete (notably for non-western populations), so the resulting species set under-represents vaginal diversity

## Evidence

- [abstract] 786 SGB threshold: [evidence span withheld — anchor status: not_found]
- [abstract] quality filter before dereplication: "yielded 18,570 prokaryotic genomes with ≥50% completeness and <5% contamination"
- [readme] SGB count in deposit: "Nonredundant prokaryotic genomes (n=786) | 440 MB | VMGC_prokaryote_SGB.tar.gz"
- [readme] input archive in deposit: "Full prokaryotic genomes (n=19,542)  | 8.1 GB | VMGC_prokaryote_MAG.tar.gz"
