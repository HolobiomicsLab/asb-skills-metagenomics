---
name: blast-based-reference-database-filtering
description: Use when a real metagenomic sample must be reduced to reads mappable
  to known references before benchmarking classifiers, and when a customized reference
  database with per-genome taxonomy must be derived from those BLAST hits.
when_to_use_negative:
- Input is a simulated metagenome with known ground truth (e.g., setA2/setB2 from
  Lindgreen et al.), where the article instead removes Eukaryote reads with unspecified
  species provenance and reads from expiring NCBI genome entries rather than BLAST-filtering
- The goal is de novo classification on unfiltered environmental reads, where discarding
  unmapped read pairs would bias the sample toward known references
- Reference databases must remain fixed across experiments, since this step derives
  the database from the sample's own BLAST hits
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: BLAST
  role: Maps reads of the real metagenome against the whole nucleotide database at
    98% sequence identity and provides the taxonomic information used to build the
    customized reference database
- name: LiME
  role: Downstream classifier run on the filtered read set and customized reference
    database (ebwt/lcp/da built with eGSA/BCR, merged with eGap)
  repo: https://github.com/veronicaguerrini/LiME
- name: eGSA
  role: Builds the ebwt, lcp and da data structures for the filtered read collection
    and customized genome set before merging
  repo: https://github.com/felipelouza/egsa
- name: eGap
  role: Merges the read-set and genome-set ebwt data structures with --lcp and --da
    options to obtain data structures for the whole collection S
  repo: https://github.com/felipelouza/egap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:03f2e615bae321751025a6f7aecacda46adba65753e64f1ab23ef2f014a6a819
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BLAST
  - CLARK-S
  - Centrifuge
  - Kraken 2
  - LiME
  - MagicBlast
  - TaxMaps
  - eGSA
  - eGap
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# blast-based-reference-database-filtering

## Summary

Pre-classification filtering of a real metagenome (SRR1804065 from the Human Microbiome Project) by mapping reads against the whole nucleotide (nt) database with BLAST at 98% sequence identity, and constructing a customized reference database from the BLAST-derived taxonomic information for downstream comparison of classifiers.

## When to use

Use when a real metagenomic sample must be reduced to reads mappable to known references before benchmarking classifiers, and when a customized reference database with per-genome taxonomy must be derived from those BLAST hits.

## When NOT to use

- Input is a simulated metagenome with known ground truth (e.g., setA2/setB2 from Lindgreen et al.), where the article instead removes Eukaryote reads with unspecified species provenance and reads from expiring NCBI genome entries rather than BLAST-filtering
- The goal is de novo classification on unfiltered environmental reads, where discarding unmapped read pairs would bias the sample toward known references
- Reference databases must remain fixed across experiments, since this step derives the database from the sample's own BLAST hits

## Inputs

- Downloaded HMP real metagenome SRR1804065 (Illumina paired-end 100 bp reads)
- Whole NCBI nucleotide sequence database for BLAST mapping
- NCBI taxonomy information for building the customized reference database

## Outputs

- Filtered read set (read pairs mapping to nt at >=98% sequence identity)
- Customized reference database (FASTA of genomes evidenced by BLAST hits)
- Taxonomic information file for the customized database (e.g., lineage file for LiME Classify)

## How to apply

Map each read pair of the downloaded HMP metagenome (SRR1804065, Illumina paired-end 100 bp) against the whole nucleotide sequence database using BLAST with a sequence identity threshold of 98%, discarding read pairs that do not map. Extract the taxonomic information from the BLAST hits to build a customized reference database containing only genomes evidenced by the filtered reads, paired with a tab-delimited lineage/taxonomy file. This customized database and filtered read set then serve as the common inputs for LiME (with ebwt/lcp/da built via eGSA/BCR and merged with eGap --lcp --da) and for Magic-BLAST, CLARK-S, Centrifuge, Kraken 2 and TaxMaps, so that tool agreement (e.g., Jaccard-based rates vs Magic-BLAST, Table S5) is measured on identical inputs. The rationale is that filtering concentrates classification on reference-supported reads, making cross-tool concordance on real data comparable rather than dominated by unmappable reads.

## Related tools

- **BLAST** (Maps reads of the real metagenome against the whole nucleotide database at 98% sequence identity and provides the taxonomic information used to build the customized reference database)
- **LiME** (Downstream classifier run on the filtered read set and customized reference database (ebwt/lcp/da built with eGSA/BCR, merged with eGap)) — https://github.com/veronicaguerrini/LiME
- **eGSA** (Builds the ebwt, lcp and da data structures for the filtered read collection and customized genome set before merging) — https://github.com/felipelouza/egsa
- **eGap** (Merges the read-set and genome-set ebwt data structures with --lcp and --da options to obtain data structures for the whole collection S) — https://github.com/felipelouza/egap

## Examples

```
blastn -query SRR1804065_reads.fasta -db nt -perc_identity 98 -outfmt '6 qseqid staxids sseqid pident' -num_threads 16 -out blast_hits.tsv
```

## Evaluation signals

- Every retained read pair has a BLAST hit with >=98% sequence identity against the nt database; unmapped pairs are discarded
- The customized reference database contains only genomes supported by BLAST-derived taxonomic assignments of the filtered reads
- Running all classifiers (LiME, Magic-BLAST, CLARK-S, Centrifuge, Kraken 2, TaxMaps) on the identical filtered read set and customized database reproduces the Table S5 agreement values (LiME vs Magic-BLAST ~95.2% identical classification, 42,788 Magic-BLAST-classified reads missed by LiME)
- LiME classification output uses the expected format C/U/A/H,IdSeqRead,TaxID,maxSim with alpha=16, beta=0.25, TaxRank=1

## Limitations

- Filtering discards read pairs without BLAST hits, so classification agreement is only measured on reference-supported reads and cannot be extended to novel organisms
- The customized reference database is sample-dependent, derived from BLAST taxonomy of the filtered reads rather than a fixed panel
- The paper does not state the dataset version/accession details beyond describing it as a downloaded dataset, so exact reproducibility depends on nt database version at the time of filtering

## Evidence

- [results] Filtering of real metagenome SRR1804065 with BLAST at 98% sequence identity: "we first filtered this dataset by using BLAST: we mapped reads against the whole nucleotide sequence database with a sequence identity of 98%"
- [discussion] BLAST-derived customized reference database construction: "we first filtered the downloaded dataset by using BLAST and extracted information to build a customized reference database"
- [results] Real metagenome dataset description: "The real metagenome SRR1804065 is a DNA tool sample of a female participant generated by using Illumina sequencing"
- [methods] Merging data structures for the collection: "To merge the two ebwts associated with the sets R and G, one could use eGap"
- [readme] LiME output format for downstream comparison: "C/U/A/H,IdSeqRead,TaxID,maxSim"
