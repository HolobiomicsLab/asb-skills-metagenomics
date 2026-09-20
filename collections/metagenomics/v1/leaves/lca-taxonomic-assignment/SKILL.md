---
name: lca-taxonomic-assignment
description: Use when classifying metagenomic HTS reads (Illumina/454 FASTQ or FASTA)
  against a protein reference database and needing taxon IDs or LCA resolution for
  ambiguous MEM matches.
when_to_use_negative:
- Reads originate from non-protein-coding genomic regions — protein-level classification
  cannot classify them (an 'obvious disadvantage' per the paper)
- The sequenced strains are already in the reference database — nucleotide-level classifiers
  are more sensitive since they can assess the entire genome
- Input sequences are already protein sequences without translation — use Kaiju's
  -p option instead of the standard translate-and-MEM workflow
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: Kaiju
  role: Performs the six-frame translation, BWT backwards search for protein-level
    MEMs, suffix-array taxon retrieval, and LCA assignment
  repo: https://github.com/bioinformatics-centre/kaiju
- name: kaiju-mkbwt
  role: Constructs the Burrows-Wheeler transform (proteins.bwt, proteins.sa) from
    the custom protein FASTA database
  repo: https://github.com/bioinformatics-centre/kaiju
- name: kaiju-mkfmi
  role: Converts the BWT into the FM-index file (.fmi) consumed by the kaiju classifier
  repo: https://github.com/bioinformatics-centre/kaiju
- name: ART
  role: Simulates verification HTS reads from whole genomes (including plasmids) to
    benchmark classification accuracy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:91564b471521bd6640fa96310e60b74b7071697c1503d121948e61dfe3c5da14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Kaiju
  - kaiju-mkbwt
  - kaiju-mkfmi
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# lca-taxonomic-assignment

## Summary

Assigns sequencing reads to NCBI taxa by translating them into six reading frames, searching for maximum exact matches (MEMs) in a protein-level BWT/FM-index database, and resolving equally long matches to multiple taxa by their least common ancestor in the taxonomic tree.

## When to use

Use when classifying metagenomic HTS reads (Illumina/454 FASTQ or FASTA) against a protein reference database and needing taxon IDs or LCA resolution for ambiguous MEM matches.

## When NOT to use

- Reads originate from non-protein-coding genomic regions — protein-level classification cannot classify them (an 'obvious disadvantage' per the paper)
- The sequenced strains are already in the reference database — nucleotide-level classifiers are more sensitive since they can assess the entire genome
- Input sequences are already protein sequences without translation — use Kaiju's -p option instead of the standard translate-and-MEM workflow

## Inputs

- FASTQ or FASTA sequencing reads (optionally gzip-compressed; paired-end supported via -i/-j)
- Protein reference database FASTA with NCBI taxon-ID headers (only uppercase standard 20 amino acids)
- NCBI taxonomy files nodes.dmp and names.dmp
- Kaiju FM-index (proteins.fmi built from proteins.bwt and proteins.sa)

## Outputs

- kaiju.out: three-column tab-separated file per read (C or U classification flag, read name, NCBI taxon identifier)
- Optional verbose columns (-v): match length/score, taxon IDs of all best-match database sequences, accession numbers, matching fragment

## How to apply

Build a Kaiju index from a FASTA protein database whose headers carry numeric NCBI taxon identifiers (optionally counter-prefixed, e.g. '>1_1358') and which contains only the uppercase 20 standard amino acids, using kaiju-mkbwt (-n 5 -a ACDEFGHIKLMNPQRSTVWY) followed by kaiju-mkfmi. Kaiju translates each read into six reading frames, splits translations at stop codons into amino acid fragments, sorts fragments by length, and queries the longest fragment first via backwards search in the BWT, enforcing a minimum match length (default m=11, supported by the observation that ~95% of shuffled-database matches are ≤11 aa). After the search, the taxon identifier is retrieved from the suffix array; if equally long matches occur in multiple taxa, the LCA is computed from nodes.dmp. Run with nodes.dmp and the .fmi index, writing Kaiju's default three-column tab-separated output (C/U flag, read name, assigned NCBI taxon identifier).

## Related tools

- **Kaiju** (Performs the six-frame translation, BWT backwards search for protein-level MEMs, suffix-array taxon retrieval, and LCA assignment) — https://github.com/bioinformatics-centre/kaiju
- **kaiju-mkbwt** (Constructs the Burrows-Wheeler transform (proteins.bwt, proteins.sa) from the custom protein FASTA database) — https://github.com/bioinformatics-centre/kaiju
- **kaiju-mkfmi** (Converts the BWT into the FM-index file (.fmi) consumed by the kaiju classifier) — https://github.com/bioinformatics-centre/kaiju
- **ART** (Simulates verification HTS reads from whole genomes (including plasmids) to benchmark classification accuracy)

## Examples

```
kaiju-mkbwt -n 5 -a ACDEFGHIKLMNPQRSTVWY -o proteins proteins.faa; kaiju-mkfmi proteins; kaiju -t nodes.dmp -f proteins.fmi -i reads.fastq -o kaiju.out -a mem
```

## Evaluation signals

- Reads simulated (e.g., with ART) from a genome present in the database are flagged C and assigned that taxon or a correct LCA ancestor; reads without adequate matches are flagged U
- Every output line has exactly three tab-separated columns with a valid C/U flag and an NCBI taxon identifier present in nodes.dmp
- No match shorter than the minimum length (default m=11 amino acids) is accepted for classification
- For reads with equally long matches to multiple taxa, the emitted taxon ID equals the LCA of those taxa in the taxonomic tree
- Low-complexity/SEG filtering stays enabled (default) to avoid spurious assignments from simple repeat patterns

## Limitations

- Protein-level classification cannot classify reads from non-protein-coding genomic regions
- Kaiju is less sensitive than nucleotide-level classifiers when the sequenced strains are in the reference database
- Genus-level classification is impossible for genera with no available reference genomes
- In Greedy mode, substitutions are only introduced at match ends because allowing them at all positions causes an exponential runtime increase with the number of allowed mismatches
- Protein sequences in the custom database may only contain the uppercase characters of the standard 20 amino acids; all other characters must be removed

## Evidence

- [discussion] Kaiju translates each read into the six possible reading frames, which are then split at stop codons into amino acid fragments.: "Kaiju translates each read into the six possible reading frames, which are then split at stop codons into amino acid fragments."
- [discussion] the taxon identifier from the corresponding database sequence is retrieved from the suffix array and printed to the output.: "the taxon identifier from the corresponding database sequence is retrieved from the suffix array and printed to the output."
- [discussion] If equally long matches are found in multiple taxa, Kaiju determines their LCA from the taxonomic tree: "If equally long matches are found in multiple taxa, Kaiju determines their LCA from the taxonomic tree"
- [readme] The taxon identifiers must be contained in the NCBI taxonomy files nodes.dmp and names.dmp.: "The taxon identifiers must be contained in the [NCBI taxonomy files](ftp://ftp.ncbi.nlm.nih.gov/pub/taxonomy/taxdump.tar.gz) nodes.dmp and names.dmp."
- [readme] Note that the protein sequences may only contain the uppercase characters of the standard 20 amino acids: "Note that the protein sequences may only contain the uppercase characters of the standard 20 amino acids, all other
characters need to be removed."
- [readme] The cutoffs for minimum required match length and match score can be changed using the options -m (default: 11) and -s (default: 65).: "The cutoffs for minimum required match length and match score can be changed using the options `-m` (default: 11) and `-s` (default: 65)."
