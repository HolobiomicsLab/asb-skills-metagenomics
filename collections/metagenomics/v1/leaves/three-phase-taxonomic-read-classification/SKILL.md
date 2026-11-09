---
name: three-phase-taxonomic-read-classification
description: Use when similarity matrices from ClusterBWT_DA (one per strand of both
  mates) must be converted into per-read-pair taxonomic assignments at species rank
  or higher, output as C/U/A/H,IdSeqRead,TaxID,maxSim lines.
when_to_use_negative:
- No similarity value for a read reaches the threshold beta, so the read must simply
  be reported as unclassified rather than pushed through taxonomic-rank fallback
- The reference database lacks a taxonomic lineage file, making higher-rank reassignment
  via lineage impossible
- Single-end data where the paired-end re-examination phase cannot add resolving information
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: LiME
  role: Provides the Classify executable that performs the three-phase read assignment
    from the ClusterBWT_DA similarity matrices
  repo: https://github.com/veronicaguerrini/LiME
- name: ClusterBWT_DA
  role: Produces the r x g similarity matrices (EBWT=1 symbol-matching variant with
    IUPAC handling, or EBWT=0 color-presence variant) that Classify consumes
  repo: https://github.com/veronicaguerrini/LiME
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:13f0f7cebf26fd66bd69c36b04dd7ecc1a662c7add434bb23ef91bb2e2b81b80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR
  - ClusterBWT_DA
  - LiME
  - eGAP
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# three-phase-taxonomic-read-classification

## Summary

LiME's Step 3 assigns each metagenomic read pair to a taxon by thresholding the r x g similarity matrices (threshold beta=0.25), selecting genomes within tolerance 0.02 of the maximum similarity, re-examining ambiguous pairs using paired-end information, and falling back to higher taxonomic ranks via the lineage file (Reference_database.csv).

## When to use

Use when similarity matrices from ClusterBWT_DA (one per strand of both mates) must be converted into per-read-pair taxonomic assignments at species rank or higher, output as C/U/A/H,IdSeqRead,TaxID,maxSim lines.

## When NOT to use

- No similarity value for a read reaches the threshold beta, so the read must simply be reported as unclassified rather than pushed through taxonomic-rank fallback
- The reference database lacks a taxonomic lineage file, making higher-rank reassignment via lineage impossible
- Single-end data where the paired-end re-examination phase cannot add resolving information

## Inputs

- Similarity matrix output files from ClusterBWT_DA for forward and reverse-complement data structures of both paired-end mates
- Total number of reads and number of reference genomes (e.g. 930 genomes in Refs.fasta)
- Lineage file Reference_database.csv (semicolon-separated taxonomic information per genome)
- TaxRank parameter (TaxRank=1 for species)

## Outputs

- Comma-separated OutputFile with lines formatted C/U/A/H,IdSeqRead,TaxID,maxSim (C=Classified, U=not classified, A=Ambiguous, H=classified at higher ranks)

## How to apply

Run Classify with the ClusterBWT_DA output files, total read and genome counts, an OutputFile, the lineage file (Reference_database.csv), TaxRank=1 (species), and a thread count. In the first phase, classify entry R_i only if its maximum similarity Mi exceeds the threshold beta (e.g. 0.25), and select as candidates all genomes that reach Mi within a tolerance of 0.02. In the second phase, re-examine ambiguous entries using paired-end information: a pair is resolved when the mates' evidence converges. Entries still unresolved are reported as Ambiguous (A) or, with HIGHER=1, reassigned at higher taxonomic ranks (up to phylum) via the taxonomic lineage before being left unclassified (U).

## Related tools

- **LiME** (Provides the Classify executable that performs the three-phase read assignment from the ClusterBWT_DA similarity matrices) — https://github.com/veronicaguerrini/LiME
- **ClusterBWT_DA** (Produces the r x g similarity matrices (EBWT=1 symbol-matching variant with IUPAC handling, or EBWT=0 color-presence variant) that Classify consumes) — https://github.com/veronicaguerrini/LiME

## Examples

```
Classify 4 mate1_F.sim mate1_RC.sim mate2_F.sim mate2_RC.sim numReads numRefs OutputFile Reference_database.csv 1 threads
```

## Evaluation signals

- Every output line conforms to the comma-separated format C/U/A/H,IdSeqRead,TaxID,maxSim
- Only read pairs whose maximum similarity exceeds beta (e.g. 0.25) appear with label C (classified)
- Ambiguous entries (A) correspond to reads with multiple genomes within tolerance 0.02 of the maximum similarity that paired-end evidence failed to resolve
- TaxIDs in the output are consistent with the lineage in Reference_database.csv at the requested TaxRank (e.g. species)
- Negative-control reads should almost never be classified: on simulated data less than 0.01% of negative control reads were classified while 99.9% of positive control reads were correctly assigned

## Limitations

- Reads whose maximum similarity does not exceed beta remain unclassified; random reads must not be assigned
- The HIGHER=1 rank-fallback mode requires the lineage file and must be set at compile time (default HIGHER=0)
- Strand direction is unknown, so both original sequences and reverse complements must be classified and reconciled; misorientation can degrade classification

## Evidence

- [methods] Threshold filtering with beta on similarity values: [evidence span withheld — anchor status: not_found]
- [methods] Selection of genomes within tolerance 0.02 of maximum similarity: [evidence span withheld — anchor status: not_found]
- [methods] Paired-end re-examination of ambiguous reads: "In this second phase, we only consider ambiguous entries."
- [readme] Output format and labels: "where C=Classified, U=not classified, A=Ambiguous, H=classified at higher ranks (if HIGHER=1)."
- [abstract] Validation of precision and specificity: "99.9% of the positive control reads are correctly assigned and the percentage of classified reads of the negative control is less than 0.01%"
