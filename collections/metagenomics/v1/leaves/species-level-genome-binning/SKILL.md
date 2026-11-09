---
name: species-level-genome-binning
description: Use when a set of quality-controlled MAGs and/or isolate genomes (FASTA)
  must be dereplicated into nonredundant species-level bins, e.g., to define 'species'
  for a microbiome reference catalogue or downstream taxonomic profiling databases.
when_to_use_negative:
- Input genomes have not passed quality filtering (e.g., below the ≥50% completeness
  / <5% contamination MIMAG-revised criteria) — clustering unfiltered bins will inflate
  SGB counts
- The goal is finer strain-level resolution rather than species-level bins — a 95%
  ANI threshold collapses strain variation
- Genomes span more than one kingdom (e.g., the VMGC's 11 fungi or 4,263 vOTUs) —
  prokaryotic ANI clustering is not appropriate for viral or eukaryotic genome dereplication
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: dRep
  role: Genome dereplication/clustering tool; run in dereplicate mode with -pa 0.9,
    -sa 0.95, -nc 0.3 to form species-level genome bins
- name: fastANI
  role: Secondary all-versus-all ANI computation engine invoked by dRep via --S_algorithm
    fastANI
- name: Kraken 2.1.3
  role: 'Downstream consumer: customized Kraken2/Bracken databases are built from
    the 786 SGBs for taxonomic profiling'
  repo: https://github.com/RChGO/VMGC/blob/main/Pipelines/kraken_MAGdb.sh
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:312379f6f289e5471014f405432f021f7d0dd0d877f213d37c66d5f466165d8e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Kraken 2.1.3
  - dRep
  - fastANI
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# species-level-genome-binning

## Summary

Dereplicates a quality-controlled prokaryotic genome collection (e.g., the VMGC's 18,570 MAGs plus 972 isolate/reference genomes) into species-level genome bins (SGBs) using 95% ANI nucleotide similarity clustering with dRep and fastANI, yielding nonredundant representative genomes (786 SGBs in VMGC).

## When to use

Use when a set of quality-controlled MAGs and/or isolate genomes (FASTA) must be dereplicated into nonredundant species-level bins, e.g., to define 'species' for a microbiome reference catalogue or downstream taxonomic profiling databases.

## When NOT to use

- Input genomes have not passed quality filtering (e.g., below the ≥50% completeness / <5% contamination MIMAG-revised criteria) — clustering unfiltered bins will inflate SGB counts
- The goal is finer strain-level resolution rather than species-level bins — a 95% ANI threshold collapses strain variation
- Genomes span more than one kingdom (e.g., the VMGC's 11 fungi or 4,263 vOTUs) — prokaryotic ANI clustering is not appropriate for viral or eukaryotic genome dereplication

## Inputs

- Quality-controlled prokaryotic genome FASTA files (VMGC: 18,570 MAGs at ≥50% completeness / <5% contamination, plus 972 isolate/reference genomes; Zenodo deposit 10457006, VMGC_prokaryote_MAG.tar.gz)
- dRep parameters: primary ANI 0.9, species ANI 0.95, -nc 0.3 coverage, fastANI as S_algorithm

## Outputs

- 786 species-level genome bins (SGBs) with genome-to-cluster assignments (sgb_clustering.tsv)
- Nonredundant representative genome FASTA set (VMGC_prokaryote_SGB.tar.gz, n=786)

## How to apply

Start from the quality-filtered genome set — in the VMGC workflow this is 19,542 prokaryotic FASTA files (18,570 MAGs passing ≥50% completeness and <5% contamination, plus 972 isolate/reference genomes) downloaded from the Zenodo deposit. Run dRep v3.4.0 in dereplicate mode with a primary ANI threshold of -pa 0.9 to precluster genomes, then species-level secondary ANI of -sa 0.95 with --S_algorithm fastANI for all-versus-all ANI computation and -nc 0.3 so the shorter genome must be covered at ≥30%. The rationale for the 95% nucleotide similarity threshold is the standard species cutoff adopted by the paper to define its 786 SGBs, which are then used to build nonredundant genome files (VMGC_prokaryote_SGB) and customized Kraken2/Bracken databases. Parse dRep's secondary clustering output to extract genome-to-cluster assignments and choose/emit cluster representatives; the expected output for VMGC is exactly 786 species-level genome bins.

## Related tools

- **dRep** (Genome dereplication/clustering tool; run in dereplicate mode with -pa 0.9, -sa 0.95, -nc 0.3 to form species-level genome bins)
- **fastANI** (Secondary all-versus-all ANI computation engine invoked by dRep via --S_algorithm fastANI)
- **Kraken 2.1.3** (Downstream consumer: customized Kraken2/Bracken databases are built from the 786 SGBs for taxonomic profiling) — https://github.com/RChGO/VMGC/blob/main/Pipelines/kraken_MAGdb.sh

## Examples

```
dRep dereplicate sgb_out -g genomes/ -pa 0.9 -sa 0.95 -nc 0.3 --S_algorithm fastANI
```

## Evaluation signals

- The number of secondary clusters must equal the reported 786 SGBs for the 19,542-genome VMGC prokaryotic set
- Cluster membership should be cross-checked against Supplementary Table 3 and the clustering shown in Fig. 2a
- Every input genome must appear in exactly one cluster in the genome-to-cluster assignment table (no unassigned or duplicated assignments)
- Each SGB should be reproducible: re-running dRep with identical -pa 0.9 / -sa 0.95 / -nc 0.3 settings on the same FASTA set yields identical cluster membership
- Median mapping rate of the 786 representative SGBs against the 4,472 source vaginal metagenomes should remain high (VMGC reports 91.7%)

## Limitations

- CheckM2 may underestimate completeness for some taxa (e.g., A. indicium and A. nucleatus), which can affect which genomes enter the clustering step
- The VMGC prokaryotic clustering depends on upstream binning; contamination from experimental operations or kits in the public datasets used to build the collection could propagate into bins
- Species boundaries defined by a flat 95% nucleotide similarity threshold may not capture complete microbial diversity, especially for under-sampled non-western vaginal populations

## Evidence

- [abstract] 786 SGB count: [evidence span withheld — anchor status: not_found]
- [abstract] quality filter: "yielded 18,570 prokaryotic genomes with ≥50% completeness and <5% contamination"
- [readme] README data table: "Nonredundant prokaryotic genomes (n=786) | 440 MB | VMGC_prokaryote_SGB.tar.gz"
- [readme] downstream SGB use: "Based on the 786 species-level genome bins (SGBs) in the VMGC, we reconstructed the prokaryotic composition of the vagina using Kraken2 and Bracken tools."
