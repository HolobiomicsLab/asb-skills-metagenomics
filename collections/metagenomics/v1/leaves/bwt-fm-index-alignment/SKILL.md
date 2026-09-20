---
name: bwt-fm-index-alignment
description: Use when classifying high-throughput sequencing reads (FASTQ/FASTA, Illumina
  or Roche/454) to NCBI taxa at the protein level, especially when k-mer classifiers
  like Kraken or Clark would need 65–78 GB of RAM but only a standard PC is available.
when_to_use_negative:
- Reads originate from non-protein-coding genomic regions — protein-level classification
  cannot assign them, so nucleotide-level classifiers are preferable
- The sequenced strains are already present in the reference database — nucleotide-level
  classifiers assessing the entire genome (e.g., as seen on the HiSeq and MiSeq data
  sets) are more sensitive
- Taxon identifiers in the custom database FASTA headers are not contained in nodes.dmp/names.dmp,
  or sequences contain characters other than the 20 uppercase standard amino acids
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: Kaiju
  role: Performs the protein-level BWT/FM-index backwards search, translating reads
    into six reading frames and assigning NCBI taxa (MEM or Greedy modes)
  repo: github.com/bioinformatics-centre/kaiju
- name: kaiju-mkbwt
  role: Constructs the Burrows-Wheeler transform of the reference protein sequences
    for the custom database workflow
  repo: github.com/bioinformatics-centre/kaiju
- name: kaiju-mkfmi
  role: Builds the FM-index (.fmi) from the BWT for use as the classification index
  repo: github.com/bioinformatics-centre/kaiju
- name: Kraken
  role: k-mer-based comparator (v0.10.4b, k=31) used to benchmark BWT-based classification
    speed and memory
- name: Clark
  role: k-mer-based comparator (v1.1.3, k=31) used to benchmark BWT-based classification
    speed and memory
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:be8447f2672fc4c52205b97bdeff7019bd89faf4786f643d9d2e0fa6bad2144d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Clark
  - Kaiju
  - Kraken
  - kaiju-mkbwt
  - kaiju-mkfmi
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# bwt-fm-index-alignment

## Summary

Protein-level metagenomic read classification by finding maximum (in-)exact matches via a backwards search in the Burrows-Wheeler transform and FM-index, as implemented in Kaiju, enabling millions of reads per minute on a standard PC with under 6 GB of memory.

## When to use

Use when classifying high-throughput sequencing reads (FASTQ/FASTA, Illumina or Roche/454) to NCBI taxa at the protein level, especially when k-mer classifiers like Kraken or Clark would need 65–78 GB of RAM but only a standard PC is available.

## When NOT to use

- Reads originate from non-protein-coding genomic regions — protein-level classification cannot assign them, so nucleotide-level classifiers are preferable
- The sequenced strains are already present in the reference database — nucleotide-level classifiers assessing the entire genome (e.g., as seen on the HiSeq and MiSeq data sets) are more sensitive
- Taxon identifiers in the custom database FASTA headers are not contained in nodes.dmp/names.dmp, or sequences contain characters other than the 20 uppercase standard amino acids

## Inputs

- FASTQ or FASTA sequencing reads (Illumina or Roche/454), single-end (-i) or paired-end (-i/-j), possibly gzip-compressed
- Reference protein database in FASTA format with numeric NCBI taxon identifiers in the headers
- NCBI taxonomy files nodes.dmp and names.dmp

## Outputs

- Tab-separated classification file (one line per read or read pair) with C/U status, read name, NCBI taxon identifier, and match length/score (with -v: all best-match taxon IDs, accession numbers, and fragment sequences)
- FM-index (.fmi) built from the BWT of the reference protein database

## How to apply

Compile Kaiju from source (cd kaiju/src; make) and build the reference index from a protein FASTA database using kaiju-mkbwt (-a ACDEFGHIKLMNPQRSTVWY) followed by kaiju-mkfmi, or use kaiju-makedb to construct the Burrows-Wheeler transform and FM-index in one step; the index requires the NCBI taxonomy files nodes.dmp and names.dmp. Kaiju translates each read into six reading frames, splits them at stop codons into amino acid fragments, sorts the fragments by length, and queries the longest fragment first against the reference using the backwards search in the BWT. Run MEM mode (-a mem) to report maximum exact matches, or Greedy mode with -e mismatches (default 3, benchmarked at 1 and 5) where MEMs of minimum seed length 7 are extended with substitutions at the left ends; filter matches by minimum length m=11, score s=65, and E-value 0.01. Use -z 25 for 25 parallel threads and -o to write the tab-separated output; if equally long matches occur in multiple taxa, Kaiju assigns the LCA taxon identifier from the taxonomic tree.

## Related tools

- **Kaiju** (Performs the protein-level BWT/FM-index backwards search, translating reads into six reading frames and assigning NCBI taxa (MEM or Greedy modes)) — github.com/bioinformatics-centre/kaiju
- **kaiju-mkbwt** (Constructs the Burrows-Wheeler transform of the reference protein sequences for the custom database workflow) — github.com/bioinformatics-centre/kaiju
- **kaiju-mkfmi** (Builds the FM-index (.fmi) from the BWT for use as the classification index) — github.com/bioinformatics-centre/kaiju
- **Kraken** (k-mer-based comparator (v0.10.4b, k=31) used to benchmark BWT-based classification speed and memory)
- **Clark** (k-mer-based comparator (v1.1.3, k=31) used to benchmark BWT-based classification speed and memory)

## Examples

```
kaiju -z 25 -t nodes.dmp -f kaiju_db.fmi -i inputfile.fastq -o kaiju.out
```

## Evaluation signals

- Output file contains one line per read with column 1 being C or U and column 3 a valid NCBI taxon identifier present in nodes.dmp
- Peak resident memory stays below 6 GB on the benchmark database (vs 72 GB for Kraken and 65–78 GB for Clark)
- Throughput on the 27.24M-read benchmark falls in the reported ranges (e.g., ~173k reads/s for Kaiju MEM on single-end 100 nt reads with -z 25)
- Multi-taxon equally long matches resolve to an LCA taxon identifier from the taxonomic tree rather than a single leaf taxon
- Index construction with kaiju-mkbwt -a ACDEFGHIKLMNPQRSTVWY followed by kaiju-mkfmi yields a proteins.fmi file that kaiju accepts via -f

## Limitations

- Protein-level classification cannot classify reads originating from non-protein-coding genomic regions
- Allowing substitutions in the backwards search causes an exponential runtime increase with the number of allowed mismatches, so Greedy restricts substitutions to match ends
- Index construction and classification memory scales with the database (e.g., refseq index needs 111 GB RAM; nr needs 219 GB), and kaiju-makedb needs ~500 GB free disk space
- In the nr database, only the first accession number per identical sequence is retained, limiting accession-level reporting for abundant proteins

## Evidence

- [abstract] novel metagenome classifier Kaiju, which finds maximum (in-)exact matches on the protein-level using the Burrows-Wheeler transform: "Here we present the novel metagenome classifier Kaiju, which finds maximum (in-)exact matches on the protein-level using the Burrows-Wheeler transform."
- [discussion] six reading frames split at stop codons: "Kaiju translates each read into the six possible reading frames, which are then split at stop codons into amino acid fragments."
- [discussion] suffix array taxon retrieval and LCA assignment: "the taxon identifier from the corresponding database sequence is retrieved from the suffix array and printed to the output."
- [methods] index construction from the reference protein database: "Before classification of reads, Kaiju's database index needs to be built from the reference protein database."
- [readme] 25-thread run command from README: "kaiju -z 25 -t nodes.dmp -f kaiju_db.fmi -i inputfile.fastq -o kaiju.out"
- [discussion] memory footprint below 6 GB: "Kaiju's memory footprint small enough (below 6 GB on our benchmark database) for running it on a standard PC"
