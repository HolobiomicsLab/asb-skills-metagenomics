---
name: adapter-and-contig-edge-kmer-filtering
description: Use when building a taxonomic k-mer database (e.g., Kraken-GB from GenBank
  draft microbial genomes) where draft contigs may contain adapter contamination or
  unreliable terminal sequence, and classification accuracy would suffer if those
  k-mers were included.
when_to_use_negative:
- The database is built from complete RefSeq genomes (standard Kraken), which do not
  require adapter or contig-edge trimming
- The goal is reduced memory usage (MiniKraken-style shrinking) rather than removing
  contaminant/edge k-mers
- Draft contigs are already adapter-trimmed and quality-trimmed at the read/assembly
  level, making the filtering redundant
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Kraken
  role: Taxonomic sequence classifier whose Kraken-GB database is built with this
    filtering applied
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter used to create the k-mer database before LCA overwrite
    and filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:2066272f34f0ce56592988afec8f3ed27b37bd22279b634b937bfe6e5cd08567
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  - Mason
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Adapter-and-contig-edge k-mer filtering

## Summary

A database-cleaning step from Kraken-GB construction: k-mers derived from known adapter sequences, plus the first and last 20 k-mers of each draft contig, are removed from the k-mer database before taxonomic labeling, so that contaminants and contig-edge artifacts do not bias classification.

## When to use

Use when building a taxonomic k-mer database (e.g., Kraken-GB from GenBank draft microbial genomes) where draft contigs may contain adapter contamination or unreliable terminal sequence, and classification accuracy would suffer if those k-mers were included.

## When NOT to use

- The database is built from complete RefSeq genomes (standard Kraken), which do not require adapter or contig-edge trimming
- The goal is reduced memory usage (MiniKraken-style shrinking) rather than removing contaminant/edge k-mers
- Draft contigs are already adapter-trimmed and quality-trimmed at the read/assembly level, making the filtering redundant

## Inputs

- GenBank draft microbial genome sequences (FASTA contigs)
- Known adapter sequence set
- Jellyfish k-mer count database with LCA-overwritten taxon IDs (NCBI taxonomy)
- Real-sequence metagenomes from SRA accessions SRS019120, SRS014468, SRS015055 (HiSeq, MiSeq) for validation

## Outputs

- Cleaned Kraken-GB k-mer database (adapters and terminal k-mers removed)
- Genus-level sensitivity/precision comparison table against the standard Kraken database

## How to apply

After counting k-mers with Jellyfish from draft microbial genomes (GenBank) and assigning LCA taxonomic IDs from the NCBI taxonomy, scan each draft contig for k-mers matching known adapter sequences and delete those k-mers from the database. Additionally remove the first and last 20 k-mers of each draft contig, since sequence at contig boundaries is more error-prone and prone to contamination. This contrasts with the standard Kraken database built from complete RefSeq genomes, which needs no such trimming. The cleaned database (Kraken-GB) is then validated by reclassifying real-sequence metagenomes (HiSeq and MiSeq reads from SRA accessions SRS019120, SRS014468, SRS015055) and confirming higher genus-level sensitivity than the standard database on those metagenomes.

## Related tools

- **Kraken** (Taxonomic sequence classifier whose Kraken-GB database is built with this filtering applied) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter used to create the k-mer database before LCA overwrite and filtering)

## Examples

```
kraken-build --db kraken-gb --add-to-library drafts.fna; kraken-build --db kraken-gb --remove-adapters-and-contig-edge-kmers --kmer-edge 20
```

## Evaluation signals

- Confirm no k-mer in the final database matches a known adapter sequence
- Confirm every draft contig contributes at most (length - 40) k-mers after removing the first and last 20 k-mers
- Kraken-GB genus-level sensitivity exceeds standard Kraken's on both the HiSeq and MiSeq real-sequence metagenomes
- Sensitivity/precision deltas replicate the reported pattern (Kraken-GB HiSeq/MiSeq sensitivity above standard Kraken)

## Limitations

- GenBank draft genomes may contain contaminating sequences from many different sources; removing adapters and edge k-mers reduces but may not eliminate such contamination
- The article does not specify a version for Jellyfish, complicating exact reproduction of the k-mer counting step
- Figure/Table numerical values (e.g., Additional file 1 Tables S1/S2, Figure 3) are referenced but not included in the extracted text, so exact sensitivity figures must come from the original figures

## Evidence

- [discussion] Kraken-GB adapter/edge k-mer removal: "we removed from the database those k-mers from known adapter sequences, as well as the first and last 20 k-mers from each of the draft contigs"
- [discussion] Draft genome contamination motivation: "some of the draft microbial genomes in GenBank contain contaminating sequences from many different sources"
- [discussion] Jellyfish k-mer database construction: "we use the Jellyfish multithreaded k-mer counter [19] to create a database"
- [other] Kraken-GB sensitivity outcome: [evidence span withheld — anchor status: not_found]
- [readme] Tool availability: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
