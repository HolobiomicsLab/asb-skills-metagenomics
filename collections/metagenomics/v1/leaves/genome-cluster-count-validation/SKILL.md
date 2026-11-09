---
name: genome-cluster-count-validation
description: Use when a paper reports a species-level genome bin count (e.g. 786 SGBs
  from 19,542 prokaryotic genomes at 95% ANI) and you must reproduce the clustering
  from the deposited genome FASTAs to verify it.
when_to_use_negative:
- Input genomes have not passed quality control — including unassembled or contaminated
  MAGs changes the cluster count; screen at >=50% completeness and <5% contamination
  first
- The goal is strain-level or genus-level dereplication rather than species-level
  95% ANI clustering
- The genome set already contains only non-redundant representatives (e.g. VMGC_prokaryote_SGB.tar.gz,
  n=786) — dereplicating those again is circular
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: dRep
  role: Dereplicate genomes into species-level bins via primary (-pa 0.9) and secondary
    (-sa 0.95) ANI clustering
- name: fastANI
  role: Secondary all-versus-all ANI computation invoked by dRep (--S_algorithm fastANI)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:1b2848ee09756d31c39d384858b77e9decc29370ad70892e5e1496644580d5de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - dRep
  - fastANI
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# genome-cluster-count-validation

## Summary

Validates a reported count of species-level genome clusters (here, 786 SGBs in the VMGC) by independently dereplicating the quality-controlled prokaryotic genomes with dRep at the 95% nucleotide similarity threshold and comparing the resulting cluster count and membership to the published bins.

## When to use

Use when a paper reports a species-level genome bin count (e.g. 786 SGBs from 19,542 prokaryotic genomes at 95% ANI) and you must reproduce the clustering from the deposited genome FASTAs to verify it.

## When NOT to use

- Input genomes have not passed quality control — including unassembled or contaminated MAGs changes the cluster count; screen at >=50% completeness and <5% contamination first
- The goal is strain-level or genus-level dereplication rather than species-level 95% ANI clustering
- The genome set already contains only non-redundant representatives (e.g. VMGC_prokaryote_SGB.tar.gz, n=786) — dereplicating those again is circular

## Inputs

- Quality-controlled prokaryotic genome FASTA files (VMGC_prokaryote_MAG.tar.gz, n=19,542: 18,570 MAGs + 972 isolates)
- Reported species-cluster count to validate (786 SGBs)
- Supplementary Table 3 cluster membership (for cross-checking)
- dRep v3.4.0 with fastANI as the secondary ANI algorithm

## Outputs

- sgb_clustering.tsv — genome-to-SGB assignment table with 786 clusters
- Validated/repudiated count of species-level genome bins
- Cluster membership comparison against Supplementary Table 3 and Fig. 2a

## How to apply

Download the quality-controlled genome FASTAs (in VMGC's case, the 18,570 MAGs passing >=50% completeness and <5% contamination plus 972 isolate/reference genomes, 19,542 total) from the deposit (https://zenodo.org/records/10457006). Run dRep v3.4.0 in dereplicate mode with primary ANI -pa 0.9, species threshold -sa 0.95, minimum shorter-genome coverage -nc 0.3, and --S_algorithm fastANI for the secondary all-versus-all ANI. Parse dRep's secondary clustering output to extract genome-to-cluster assignments and emit the bins (e.g. sgb_clustering.tsv). Confirm the number of clusters matches the reported 786 SGBs and cross-check membership against Supplementary Table 3 and Fig. 2a; a mismatch in count usually indicates a divergence in threshold, coverage parameter, or the exact genome set included.

## Related tools

- **dRep** (Dereplicate genomes into species-level bins via primary (-pa 0.9) and secondary (-sa 0.95) ANI clustering)
- **fastANI** (Secondary all-versus-all ANI computation invoked by dRep (--S_algorithm fastANI))

## Examples

```
dRep dereplicate sgb_out -g genomes/*.fasta -pa 0.9 -sa 0.95 -nc 0.3 --S_algorithm fastANI
```

## Evaluation signals

- Number of parsed clusters equals the reported 786 SGBs
- Total genomes clustered equals the input set (19,542 = 18,570 MAGs + 972 isolates) with no genomes dropped or double-assigned
- Each SGB's members share >=95% ANI (dRep secondary threshold) and clusters are separated by <90% primary ANI linkage
- Genome-to-cluster assignments agree with the article's Supplementary Table 3 and the clustering shown in Fig. 2a
- Cluster count is stable when re-run with a fixed seed/deterministic fastANI parameters

## Limitations

- The CheckM2-based completeness estimates used for pre-clustering quality control may be underestimated for some taxa (e.g. A. indicium and A. nucleatus), so borderline genomes could be wrongly excluded before clustering
- Genome set composition is decisive: including or excluding even a few deposited genomes shifts the cluster count away from 786
- dRep version and fastANI parameter drift across releases can change secondary clustering results, so version pinning (dRep v3.4.0) is required for exact reproduction

## Evidence

- [methods] Run dRep v3.4.0 in dereplicate mode with -pa 0.9 (primary ANI), -sa 0.95 (species-level ANI threshold), -nc 0.3: [evidence span withheld — anchor status: not_found]
- [abstract] resulting in 786 species-level genome bins (SGBs, referred to as 'species' thereafter) based on the 95% nucleotide similarity threshold: [evidence span withheld — anchor status: not_found]
- [readme] Nonredundant prokaryotic genomes (n=786) | 440 MB | VMGC_prokaryote_SGB.tar.gz: "Nonredundant prokaryotic genomes (n=786) | 440 MB | VMGC_prokaryote_SGB.tar.gz"
- [abstract] yielded 18,570 prokaryotic genomes with >=50% completeness and <5% contamination: [evidence span withheld — anchor status: not_found]
