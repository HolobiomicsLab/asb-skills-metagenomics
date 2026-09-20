---
name: six-frame-translation-protein-search
description: Use when classifying whole-genome shotgun metagenomic reads (Illumina
  FASTQ/FASTA) against protein reference databases, especially when taxa are underrepresented
  in the database or divergence prevents exact k-mer nucleotide matches.
when_to_use_negative:
- Reads originate from non-protein-coding genomic regions — protein-level classification
  cannot assign them
- Sequenced strains are already in the reference database — nucleotide-level classifiers
  (e.g. Kraken) will be more sensitive since they can assess the entire genome
- Input is already protein sequence without enabling option -p to disable translation
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: Kaiju
  role: Performs the six-frame translation, stop-codon splitting, BWT backwards search,
    and taxon assignment of reads against the protein FM-index
  repo: https://github.com/bioinformatics-centre/kaiju
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:e2516ee6870d2837d09b786877c53450b78ef6f50430dd4c37789afcb7fe9282
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - Kaiju
  - Kraken
  - R
  - kaiju-mergeOutputs
  - kaiju2table
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# Six-frame translation protein search

## Summary

Kaiju's core classification method: translate each NGS read into six reading frames, split at stop codons into amino acid fragments, and query maximum (in-)exact protein matches via backwards search in the BWT to assign NCBI taxonomy. This protein-level approach classifies reads that nucleotide k-mer classifiers miss.

## When to use

Use when classifying whole-genome shotgun metagenomic reads (Illumina FASTQ/FASTA) against protein reference databases, especially when taxa are underrepresented in the database or divergence prevents exact k-mer nucleotide matches.

## When NOT to use

- Reads originate from non-protein-coding genomic regions — protein-level classification cannot assign them
- Sequenced strains are already in the reference database — nucleotide-level classifiers (e.g. Kraken) will be more sensitive since they can assess the entire genome
- Input is already protein sequence without enabling option -p to disable translation

## Inputs

- Whole-genome shotgun metagenomic reads in FASTQ or FASTA (gzip-compressed allowed), paired-end via -i/-j
- Kaiju index (.fmi) built from a protein reference database (e.g. RefSeq, nr, or custom genome FASTA converted to proteins)
- NCBI taxonomy files nodes.dmp and names.dmp

## Outputs

- Per-read classification output file (e.g. kaiju.out) with columns: C/U status, read name, assigned NCBI taxon identifier (optionally match score/length with -v)
- Optionally rank-level summary tables via kaiju2table/Krona visualizations

## How to apply

First build the Kaiju FM-index (.fmi) from reference proteins using kaiju-makedb or kaiju-mkbwt/kaiju-mkfmi on a custom FASTA whose headers are numeric NCBI taxon IDs; taxon IDs must be present in nodes.dmp/names.dmp. Kaiju translates each read into the six possible reading frames, splits them at stop codons into amino acid fragments, sorts fragments by length, and queries the longest fragment first with a backwards search in the BWT; the taxon is retrieved from the suffix array, with LCA assigned for equally long matches in multiple taxa. Run in MEM mode (-a mem, -m 12) for exact maximum matches, or Greedy mode (-e mismatches, -s score, defaults m=11 and s=65) for substitutions at match ends; keep SEG low-complexity filtering enabled (-x, on by default) to avoid spurious assignments. Use kaiju or kaiju-multi with -z for threads and -o for output files; only uppercase standard 20 amino acid characters are allowed in custom databases.

## Related tools

- **Kaiju** (Performs the six-frame translation, stop-codon splitting, BWT backwards search, and taxon assignment of reads against the protein FM-index) — https://github.com/bioinformatics-centre/kaiju

## Examples

```
kaiju -z 25 -t nodes.dmp -f kaiju_db.fmi -i reads.fastq -a mem -m 12 -x -o kaiju.out
```

## Evaluation signals

- Output file contains one line per read (or read pair) with C/U flag and a NCBI taxon identifier valid in nodes.dmp
- In sensitivity benchmarks, Kaiju classifies on average about twice as many reads as Kraken across the 10 real metagenomic datasets
- Memory footprint stays below ~6 GB on the benchmark database, confirming the index was built correctly
- Enabling SEG filtering (-x) removes false-positive taxon assignments caused by simple repeat patterns or sequencing noise
- In the genome-exclusion benchmark, sensitivity should exceed Kraken/Clark with only little loss of precision

## Limitations

- Protein-level classification cannot classify reads originating from non-protein-coding genomic regions
- Allowing substitutions in the backwards search causes an exponential runtime increase with the number of allowed mismatches
- The greedy heuristic restricts substitutions to the ends of matches rather than all query positions, to limit search-space expansion
- Read names are truncated after '/' or space, which can affect paired-end name matching between files

## Evidence

- [discussion] Six-frame translation and stop-codon fragment splitting: "Kaiju translates each read into the six possible reading frames, which are then split at stop codons into amino acid fragments."
- [discussion] BWT backwards search with LCA resolution: "If equally long matches are found in multiple taxa, Kaiju determines their LCA from the taxonomic tree"
- [readme] SEG low-complexity filtering option: "Option `-x` enables filtering of query sequences containing low-complexity regions by using the SEG algorithm from the blast+ package."
- [readme] MEM run mode selection: "The run mode can be changed to **MEM** using option `-a`:"
- [abstract] Higher sensitivity on real metagenomes: "We also demonstrate that Kaiju classifies up to 10 times more reads in real metagenomes."
