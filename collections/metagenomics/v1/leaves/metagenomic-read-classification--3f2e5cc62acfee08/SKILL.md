---
name: metagenomic-read-classification
description: Use when assigning taxonomic labels to large sets of metagenomic reads
  (FASTQ/FASTA from SRA accessions such as SRS019120, SRS014468, SRS015055, or Mason-simulated
  reads), or when benchmarking classifier speed and accuracy against Megablast or
  MetaPhlAn.
when_to_use_negative:
- When the goal is gene content estimation requiring classification of nearly every
  read with abundance estimation — abundance programs like MetaPhlAn classify only
  a small percentage of sequences and serve a different purpose
- When reads come from organisms whose clade of origin is absent from the reference
  database — clade exclusion experiments show sensitivity can drop to roughly 33%
  when the species was never seen
- When the input is already a taxonomy/abundance profile rather than raw sequencing
  reads
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Kraken
  role: 'Primary classifier: assigns taxonomic labels via exact k-mer LCA matches
    and root-to-leaf path scoring in a classification tree'
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter used to create the Kraken database; its 4-byte
    count spaces are overwritten with LCA taxonomic IDs
- name: Mason
  role: Read simulator used to generate the simBA-5 metagenome with its Illumina model,
    default mismatch/indel rates multiplied by five
- name: Megablast
  role: Fastest BLAST mode used as the speed and accuracy comparison baseline (Kraken
    is 909x faster)
- name: MetaPhlAn
  role: Abundance estimation program used as a secondary speed comparison (Kraken
    is 11x faster)
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_005
  - task_006
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:69ba2eb1c731e10df4f95fa77cc5916fc57b9f5015f2f61bda0c1e84d4db22fe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  - Mason
  - Megablast
  - MetaPhlAn
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metagenomic-read-classification

## Summary

Assign taxonomic labels to metagenomic DNA sequences with Kraken by querying a Jellyfish-derived k-mer LCA database with exact k-mer matches, achieving Megablast-comparable accuracy at millions of reads per minute. It covers database construction (RefSeq genomes, NCBI taxonomy), paired-read concatenation, and genus-level sensitivity/precision evaluation.

## When to use

Use when assigning taxonomic labels to large sets of metagenomic reads (FASTQ/FASTA from SRA accessions such as SRS019120, SRS014468, SRS015055, or Mason-simulated reads), or when benchmarking classifier speed and accuracy against Megablast or MetaPhlAn.

## When NOT to use

- When the goal is gene content estimation requiring classification of nearly every read with abundance estimation — abundance programs like MetaPhlAn classify only a small percentage of sequences and serve a different purpose
- When reads come from organisms whose clade of origin is absent from the reference database — clade exclusion experiments show sensitivity can drop to roughly 33% when the species was never seen
- When the input is already a taxonomy/abundance profile rather than raw sequencing reads

## Inputs

- Metagenomic read sets in FASTQ or FASTA (e.g., HiSeq and MiSeq metagenomes from SRA accessions SRS019120, SRS014468, SRS015055)
- Mason-simulated read sets (simBA-5 with 2% average mismatch rate; simMC subset of 3,333 reads over 31 genera)
- Complete NCBI RefSeq bacterial, archaeal, viral, and human genomes for database construction
- NCBI taxonomy database (taxon IDs and clade information)

## Outputs

- Per-read taxonomic classification output from Kraken (read ID to LCA taxon assignment)
- Kraken reports per accession with fractions of unclassified reads and genus-level distributions (e.g., hmp_saliva_kraken_report.tsv)
- Benchmark tables of reads-per-minute and speedup factors over Megablast (speed_table.csv)
- Accuracy tables of genus-level sensitivity/precision for Kraken, MiniKraken, -Q mode, and Kraken-GB (accuracy_table.csv, table2_clade_exclusion.csv)

## How to apply

Build the database by counting k-mers from complete NCBI RefSeq microbial genomes with the Jellyfish multithreaded k-mer counter, then overwrite the 4-byte count spaces with the k-mer LCA taxonomic IDs from the NCBI taxonomy, computing the LCA of the stored and new taxon on conflicts. Classify each read by collecting its k-mers, mapping each to the LCA taxon, building a weighted classification tree, and scoring root-to-leaf paths; for paired-end data, concatenate mates with 'NNNNN' so each pair is classified as a single unit (ambiguous-nucleotide k-mers are ignored). For speed benchmarks, read all database files into memory three times before timing to populate the OS cache, then record reads-per-minute on HiSeq, simBA-5 (Mason with default mismatch/indel rates multiplied by five, average mismatch rate 2%), and MiSeq metagenomes, comparing against Megablast. For accuracy evaluation, compute genus-level sensitivity and precision, excluding reads whose origin lacks a taxonomic entry at the measured or excluded rank or whose excluded-rank clade is not shared by at least two other represented taxa. For low-memory settings, build MiniKraken 4-GB by removing the first 18 of every block of 19 records from the standard database.

## Related tools

- **Kraken** (Primary classifier: assigns taxonomic labels via exact k-mer LCA matches and root-to-leaf path scoring in a classification tree) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter used to create the Kraken database; its 4-byte count spaces are overwritten with LCA taxonomic IDs)
- **Mason** (Read simulator used to generate the simBA-5 metagenome with its Illumina model, default mismatch/indel rates multiplied by five)
- **Megablast** (Fastest BLAST mode used as the speed and accuracy comparison baseline (Kraken is 909x faster))
- **MetaPhlAn** (Abundance estimation program used as a secondary speed comparison (Kraken is 11x faster))

## Evaluation signals

- Throughput thresholds: Kraken should exceed ~1.5M reads/min on the HiSeq metagenome, ~1.3M rpm on simBA-5, and ~890K rpm on the MiSeq metagenome (over 4.1M rpm in its fastest mode), with 150-240x speedup over Megablast
- HMP saliva classification: approximately 68.2% of reads unclassified, with genus distribution among classified reads of Streptococcus 30%, Haemophilus 17%, Prevotella 13% (values within a few percentage points accepted)
- Clade exclusion validation: every measured/excluded rank pair must achieve precision >= 93%, with best-case sensitivity of roughly 33% when the species clade is excluded
- Variant database checks: MiniKraken sensitivity drops ~11% on real-sequence metagenomes and >25 points on simBA-5, while MiniKraken precision is at or above standard Kraken's on all three metagenomes
- Database integrity: k-mer count spaces correctly overwritten with LCA taxon IDs, with LCA recomputed on conflicts between stored and new sequence taxon

## Limitations

- Sensitivity drops dramatically for novel organisms: at best approximately 33% of reads classified when the species clade of origin was excluded from the database
- Some organisms cannot be correctly identified at genus level (e.g., Pelosinus fermentans in the HiSeq metagenome; Proteus vulgaris in the MiSeq metagenome is often classified incorrectly)
- MiniKraken's reduced-memory design trades sensitivity for precision: sensitivity drops ~11% on real-sequence metagenomes and >25 points on the high-error simBA-5 metagenome
- Kraken ignores k-mers with ambiguous nucleotides, so k-mers spanning 'N' characters (including the NNNNN paired-mate junction) do not affect classification
- GenBank draft genomes can contain contaminating sequences, motivating the Kraken-GB variant that removes adapter k-mers and the first/last 20 k-mers of each draft contig

## Evidence

- [abstract] Kraken is an ultrafast and highly accurate program for assigning taxonomic labels to metagenomic DNA sequences.: "Kraken is an ultrafast and highly accurate program for assigning taxonomic labels to metagenomic DNA sequences."
- [abstract] Using exact alignment of k-mers, Kraken achieves classification accuracy comparable to the fastest BLAST program.: "Using exact alignment of k-mers, Kraken achieves classification accuracy comparable to the fastest BLAST program."
- [methods] the 4-byte spaces Jellyfish used to store the k-mer counts in the database file are instead used by Kraken to store the taxonomic ID numbers: [evidence span withheld — anchor status: not_found]
- [methods] we multiplied the default mismatch and indel rates by five, resulting in an average mismatch rate of 2%: "we multiplied the default mismatch and indel rates by five, resulting in an average mismatch rate of 2%"
- [methods] we removed the first 18 of every block of 19 records in the standard Kraken database: "we removed the first 18 of every block of 19 records in the standard Kraken database"
- [methods] we joined the reads together by concatenating the mates with a sequence of 'NNNNN' between them: [evidence span withheld — anchor status: not_found]
- [readme] Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken.: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
