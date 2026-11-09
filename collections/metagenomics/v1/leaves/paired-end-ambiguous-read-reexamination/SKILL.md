---
name: paired-end-ambiguous-read-reexamination
description: Use when paired-end metagenomic reads (e.g. 100 bp Illumina mates) remain
  ambiguous after similarity scoring against reference genomes and mate-consistency
  should resolve them or reassign them to a higher taxonomic rank.
when_to_use_negative:
- Input is single-end reads with no mate information, since the re-examination phase
  relies on paired-end consistency to break ties
- Read pairs whose maximum similarity score does not exceed beta — these are reported
  unclassified and never enter the ambiguous re-examination phase
- Classification targets requiring assembly or alignment of reads rather than the
  alignment-free eBWT similarity framework
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: LiME
  role: Provides the Classify step that performs threshold filtering (beta), tolerance-0.02
    genome selection, paired-end re-examination of ambiguous entries, and lineage-based
    higher-rank fallback
  repo: https://github.com/veronicaguerrini/LiME
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:64bdf1620acb1a714d185a885b8d51688c8ec0f483c31f6de81fcecf65f0f0ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR
  - LiME
  - eGAP
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# paired-end-ambiguous-read-reexamination

## Summary

Second- and third-phase classification logic of LiME in which entries whose maximum similarity score exceeds threshold beta but map to several genomes (within tolerance 0.02 of the maximum) are re-examined using paired-end mate information, with fallback to higher taxonomic ranks via the lineage file when ambiguity persists.

## When to use

Use when paired-end metagenomic reads (e.g. 100 bp Illumina mates) remain ambiguous after similarity scoring against reference genomes and mate-consistency should resolve them or reassign them to a higher taxonomic rank.

## When NOT to use

- Input is single-end reads with no mate information, since the re-examination phase relies on paired-end consistency to break ties
- Read pairs whose maximum similarity score does not exceed beta — these are reported unclassified and never enter the ambiguous re-examination phase
- Classification targets requiring assembly or alignment of reads rather than the alignment-free eBWT similarity framework

## Inputs

- ClusterBWT_DA output similarity-matrix files (forward and reverse-complement data structures for both paired-end mates)
- Total number of reads and number of reference genomes (e.g. 930 genomes in Refs.fasta)
- Lineage file Reference_database.csv (semicolon-separated taxonomic lineage per genome)
- TaxRank value (e.g. 1 for species-level classification)

## Outputs

- Comma-separated OutputFile with lines in the format C/U/A/H,IdSeqRead,TaxID,maxSim
- Read-pair assignments where ambiguous pairs are resolved by mate information or labeled H (higher-rank) / U (unclassified)

## How to apply

After ClusterBWT_DA computes the r x g similarity matrices for the forward and reverse-complement data structures of both mates, Classify attempts to assign each read pair only if the maximum similarity exceeds beta (0.25 in the reference workflow for 100 bp paired-end reads); genomes that approximately reach the maximum Mi with tolerance 0.02 are selected. In the second phase, only ambiguous entries are considered, and the pairing information of the two mates is used to disambiguate the assignment. If the pair is still ambiguous, classification falls back to higher taxonomic ranks using the semicolon-separated lineage file (Reference_database.csv) at the chosen TaxRank (e.g. 1 for species). Because strand direction is unknown, the procedure is applied to both the original sequences and their reverse complements so reads stay properly oriented.

## Related tools

- **LiME** (Provides the Classify step that performs threshold filtering (beta), tolerance-0.02 genome selection, paired-end re-examination of ambiguous entries, and lineage-based higher-rank fallback) — https://github.com/veronicaguerrini/LiME

## Examples

```
Classify N Input1 ... InputN numReads numRefs OutputFile LineageFile TaxRank threads
```

## Evaluation signals

- Every output line conforms to the comma-separated format C/U/A/H,IdSeqRead,TaxID,maxSim
- Only read pairs whose maximum similarity score exceeds beta are reported as classified (C); lower-scoring pairs appear as U
- Genomes retained for an ambiguous entry all lie within tolerance 0.02 of the maximum similarity value Mi
- Ambiguous pairs surviving mate re-examination are relabeled with a higher taxonomic rank consistent with the lineage file and TaxRank setting
- Both strands (forward and reverse complement) of both mates contribute, so each read pair is evaluated against four similarity matrices

## Limitations

- The re-examination cannot resolve pairs whose mates are also inconsistent or ambiguous; classification then depends on lineage-based higher-rank fallback (HIGHER=1 build option)
- Similarity scoring treats IUPAC ambiguity codes as matching themselves or associated letters, which can widen the set of genomes within the 0.02 tolerance
- Without computing reverse-complement data structures, reads in the wrong orientation lose similarity signal and may be misclassified or left ambiguous

## Evidence

- [methods] Re-examination of ambiguous reads using paired-end information: "Re-examination of ambiguous reads– In this second phase, we only consider ambiguous entries."
- [methods] Genome selection tolerance of 0.02 around the maximum similarity: [evidence span withheld — anchor status: not_found]
- [methods] Threshold beta gates whether a read is classified at all: [evidence span withheld — anchor status: not_found]
- [readme] Reverse complements required for paired-end orientation handling: "since reads' strand direction is unknown, both the original sequences and their reverse complements should be considered"
- [readme] Output labels including the Ambiguous state: "where C=Classified, U=not classified, A=Ambiguous, H=classified at higher ranks (if HIGHER=1)."
