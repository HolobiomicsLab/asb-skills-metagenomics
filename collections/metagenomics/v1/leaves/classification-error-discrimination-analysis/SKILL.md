---
name: classification-error-discrimination-analysis
description: Decision support for choosing a minimum match length (or similar score
  cutoff) so that random short spurious matches are excluded while classification
  sensitivity is preserved.
when_to_use_negative:
- You need sensitivity rather than cutoff calibration — the article notes Kaiju is
  less sensitive than nucleotide-level classifiers when sequenced strains are present
  in the reference database.
- Reads originate from non-protein-coding genomic regions, which protein-level Kaiju
  classification cannot assign regardless of the cutoff.
- The classifier's match lengths are not comparable across runs (e.g. k-mer classifiers
  like Kraken/Clark use fixed k=31 rather than variable-length MEMs).
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: Classifies simulated reads in MEM mode (-a mem) against original and shuffled
    database indexes; verbose output supplies match lengths
  repo: github.com/bioinformatics-centre/kaiju
- name: uShuffle
  role: Generates the composition-preserving shuffled null protein database (window
    length 100 aa)
- name: ART
  role: Simulates HTS metagenomic reads used as query sequences for both classification
    runs
- name: kaiju-mkbwt
  role: Builds the Burrows-Wheeler transform index from original and shuffled protein
    sets
  repo: github.com/bioinformatics-centre/kaiju
- name: kaiju-mkfmi
  role: Converts the BWT into the FM-index used by Kaiju
  repo: github.com/bioinformatics-centre/kaiju
- name: R
  role: Statistical environment for tabulating and analyzing match-length distributions
- name: ggplot2
  role: Plots the match-length distributions reproducing Supplementary Fig. 5
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:7bb2c67b8d90d9019a7f37be9ad3420de6d7d70618998eb3d31f301a12f2813c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ART
  - Kaiju
  - R
  - ggplot2
  - kaiju-mkbwt
  - kaiju-mkfmi
  - uShuffle
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# classification-error-discrimination-analysis

## Summary

Empirically calibrates a classifier's minimum match length cutoff by classifying simulated reads against a composition-preserving shuffled null database and comparing match-length distributions of true versus spurious matches, as done to justify Kaiju's default m=11 in MEM mode.

## When to use

Decision support for choosing a minimum match length (or similar score cutoff) so that random short spurious matches are excluded while classification sensitivity is preserved.

## When NOT to use

- You need sensitivity rather than cutoff calibration — the article notes Kaiju is less sensitive than nucleotide-level classifiers when sequenced strains are present in the reference database.
- Reads originate from non-protein-coding genomic regions, which protein-level Kaiju classification cannot assign regardless of the cutoff.
- The classifier's match lengths are not comparable across runs (e.g. k-mer classifiers like Kraken/Clark use fixed k=31 rather than variable-length MEMs).

## Inputs

- Microbial subset of the NCBI nr protein database (FASTA)
- ART-simulated metagenomic sequencing reads (FASTQ)
- NCBI taxonomy files nodes.dmp and names.dmp

## Outputs

- Shuffled-database Kaiju index (.bwt/.fmi files)
- Verbose Kaiju classification outputs with match lengths in column 4
- Match-length distributions (Supplementary Fig. 5) supporting the m=11 default

## How to apply

Create a null reference by shuffling the microbial subset of the NCBI nr protein database with uShuffle using a window length of 100 amino acids to preserve local amino acid composition. Build Kaiju indexes for both shuffled and original protein sets with kaiju-mkbwt (alphabet ACDEFGHIKLMNPQRSTVWY) followed by kaiju-mkfmi, then classify ART-simulated metagenomic reads against each index in MEM mode (-a mem) with the default minimum match length m=11. Extract best-match lengths from column 4 of the verbose Kaiju output, tabulate length distributions separately for matches to the shuffled database and for correct versus wrong classifications against the original database, and plot with ggplot2 in R. Use the shuffled-database distribution to set the cutoff: ~95% of shuffled matches are ≤11 amino acids, whereas wrong classifications on the original database show a much heavier tail above 11 aa than correct ones (~2%), justifying m=11; test neighboring values (e.g. m=12) to confirm precision loss outweighs sensitivity gained.

## Related tools

- **Kaiju** (Classifies simulated reads in MEM mode (-a mem) against original and shuffled database indexes; verbose output supplies match lengths) — github.com/bioinformatics-centre/kaiju
- **uShuffle** (Generates the composition-preserving shuffled null protein database (window length 100 aa))
- **ART** (Simulates HTS metagenomic reads used as query sequences for both classification runs)
- **kaiju-mkbwt** (Builds the Burrows-Wheeler transform index from original and shuffled protein sets) — github.com/bioinformatics-centre/kaiju
- **kaiju-mkfmi** (Converts the BWT into the FM-index used by Kaiju) — github.com/bioinformatics-centre/kaiju
- **R** (Statistical environment for tabulating and analyzing match-length distributions)
- **ggplot2** (Plots the match-length distributions reproducing Supplementary Fig. 5)

## Examples

```
kaiju -t nodes.dmp -f kaiju_db.fmi -i reads.fastq -a mem -v -o kaiju_mem.out
```

## Evaluation signals

- Approximately 95% of matches against the shuffled database are ≤11 amino acids long.
- More than 75% of wrong classifications against the original database have matches ≤11 aa, versus only ~2% of correct classifications.
- Raising the cutoff from m=11 to m=12 loses more precision than the sensitivity gained, on all data sets at both genus and phylum level.
- The shuffled and original indexes are built with identical parameters (alphabet ACDEFGHIKLMNPQRSTVWY) so match-length distributions are directly comparable.
- Column 4 of the verbose Kaiju output yields a defined best-match length for every classified read in both runs.

## Limitations

- The shuffled-database null preserves local composition but is derived from a single database snapshot (NR microbial subset, genomes from 16 December 2014); cutoffs may shift for other reference databases.
- Protein-level classification cannot classify reads from non-protein-coding genomic regions, capping achievable sensitivity at any cutoff.
- Allowing substitutions in the backwards search causes an exponential runtime increase with the number of allowed mismatches, so this calibration is done in exact-match MEM mode.

## Evidence

- [discussion] Shuffled database calibration of m=11: "we created a shuffled version of the microbial subset of NCBI's NR protein database, using uShuffle25 with a window length of 100 amino acids"
- [discussion] Shuffled matches ≤11 aa: "one can observe that ∼95% of them are ≤11 amino acids long"
- [discussion] Greedy-mode score default: "we chose s=65 as default value for Kaiju's Greedy mode"
- [readme] MEM mode invocation: "The run mode can be changed to **MEM** using option `-a`:"
- [readme] Verbose output column 4: "4. the length or score of the best match used for classification"
- [readme] Index construction commands: "kaiju-mkbwt -n 5 -a ACDEFGHIKLMNPQRSTVWY -o proteins proteins.faa
kaiju-mkfmi proteins"
