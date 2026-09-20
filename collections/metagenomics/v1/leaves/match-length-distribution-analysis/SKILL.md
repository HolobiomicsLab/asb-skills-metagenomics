---
name: match-length-distribution-analysis
description: Use when choosing or validating a minimum match length (or similar alignment-length)
  cutoff for a classifier by comparing match-length distributions on real versus shuffled
  reference databases.
when_to_use_negative:
- When a nucleotide-level classifier is appropriate because the sequenced strains
  are present in the reference database — Kaiju's protein-level matching is then less
  sensitive since non-protein-coding regions cannot be classified
- When the goal is classification accuracy benchmarking (genome exclusion) rather
  than calibrating a match-length parameter — use the 882-genome/242-genus exclusion
  benchmark workflow instead
- When reads are already protein sequences and only need scoring parameter tuning
  (Greedy mode score s=65), since this skill targets MEM-mode match length
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: Classifies simulated reads in MEM mode against original and shuffled protein
    databases; mkbwt/mkfmi build the BWT/FM-index; verbose output exposes match lengths
  repo: https://github.com/bioinformatics-centre/kaiju
- name: uShuffle
  role: Generates the shuffled null-model protein database preserving local amino
    acid composition (window length 100 aa)
- name: ART
  role: Simulates HTS reads from genomes so ground-truth correct/wrong classifications
    are known
- name: R
  role: Statistical software in which match-length distributions are tabulated and
    plotted
- name: ggplot2
  role: R package used to plot the match-length distributions (Supplementary Fig.
    5)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:0a494379bdc8f5574acb0dab71afd2628f08469c0e1eca82189fc8d1638a12c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Kaiju
  - R
  - ggplot2
  - uShuffle
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# match-length-distribution-analysis

## Summary

Empirically calibrates a protein-level metagenome classifier's minimum match length cutoff by classifying simulated reads against both an original and a composition-preserving shuffled reference database, then comparing match-length distributions to pick a cutoff (Kaiju MEM mode: m=11) that excludes random spurious matches while preserving sensitivity.

## When to use

Use when choosing or validating a minimum match length (or similar alignment-length) cutoff for a classifier by comparing match-length distributions on real versus shuffled reference databases.

## When NOT to use

- When a nucleotide-level classifier is appropriate because the sequenced strains are present in the reference database — Kaiju's protein-level matching is then less sensitive since non-protein-coding regions cannot be classified
- When the goal is classification accuracy benchmarking (genome exclusion) rather than calibrating a match-length parameter — use the 882-genome/242-genus exclusion benchmark workflow instead
- When reads are already protein sequences and only need scoring parameter tuning (Greedy mode score s=65), since this skill targets MEM-mode match length

## Inputs

- Microbial subset of the NCBI nr protein database (FASTA protein sequences with NCBI taxon ID headers)
- ART-simulated metagenomic sequencing reads (FASTQ) from known genomes
- NCBI taxonomy files nodes.dmp and names.dmp

## Outputs

- Kaiju index files (.bwt and .fmi) for original and shuffled protein databases
- Verbose Kaiju classification output with best-match length in column 4 for both database runs
- Match-length distribution plots (ggplot2/R), separating shuffled-database matches and wrong vs. correct classifications, reproducing Supplementary Fig. 5

## How to apply

Create a null model by shuffling the microbial subset of the NCBI nr protein database with uShuffle using a window length of 100 amino acids to preserve local amino acid composition. Build Kaiju indexes for both the shuffled and original protein sets with kaiju-mkbwt (alphabet ACDEFGHIKLMNPQRSTVWY) followed by kaiju-mkfmi. Classify the same ART-simulated metagenomic reads against both indexes in MEM mode (-a mem, default m=11), extract best-match lengths from column 4 of the verbose (-v) output, and tabulate length distributions separately for wrong versus correct classifications in the original-database run. Plot the distributions with ggplot2 in R; choose the cutoff where spurious (shuffled-database) matches disappear — here ~95% of shuffled-database matches are ≤11 amino acids, so m=11 is the default — and check that raising it (e.g., 11→12) does not trade favorably between precision and sensitivity across genus- and phylum-level classifications.

## Related tools

- **Kaiju** (Classifies simulated reads in MEM mode against original and shuffled protein databases; mkbwt/mkfmi build the BWT/FM-index; verbose output exposes match lengths) — https://github.com/bioinformatics-centre/kaiju
- **uShuffle** (Generates the shuffled null-model protein database preserving local amino acid composition (window length 100 aa))
- **ART** (Simulates HTS reads from genomes so ground-truth correct/wrong classifications are known)
- **R** (Statistical software in which match-length distributions are tabulated and plotted)
- **ggplot2** (R package used to plot the match-length distributions (Supplementary Fig. 5))

## Examples

```
kaiju -t nodes.dmp -f shuffled.fmi -i reads.fastq -a mem -m 11 -v -o shuffled.out
```

## Evaluation signals

- Approximately 95% of matches against the shuffled database are ≤11 amino acids long
- In the original-database run, >75% of wrong classifications but only ~2% of correct classifications have matches ≤11 aa
- Raising the cutoff from m=11 to m=12 loses more precision than the sensitivity gained, in all data sets at both genus and phylum level (confirming m=11)
- Shuffled database must preserve local composition: uShuffle window length of 100 amino acids is used
- Both indexes built with identical alphabet ACDEFGHIKLMNPQRSTVWY so match lengths are comparable

## Limitations

- Protein-level classification cannot classify reads originating from non-protein-coding genomic regions
- Kaiju is less sensitive than nucleotide-level classifiers when sequenced strains are present in the reference database
- Conclusions are specific to the simulated read sets and database snapshot used (NCBI nr microbial subset; genomes snapshot 16 December 2014); the cutoff should be re-validated for other databases or read types
- SEG filtering (enabled by default via -x) is recommended to avoid false positive assignments from spurious matches due to simple repeat patterns

## Evidence

- [discussion] one can observe that ∼95% of them are ≤11 amino acids long: "one can observe that ∼95% of them are ≤11 amino acids long"
- [discussion] we created a shuffled version of the microbial subset of NCBI's NR protein database, using uShuffle25 with a window length of 100 amino acids: "we created a shuffled version of the microbial subset of NCBI's NR protein database, using uShuffle25 with a window length of 100 amino acids"
- [readme] kaiju-mkbwt alphabet command: "kaiju-mkbwt -n 5 -a ACDEFGHIKLMNPQRSTVWY -o proteins proteins.faa
kaiju-mkfmi proteins"
- [readme] The cutoffs for minimum required match length and match score can be changed using the options `-m` (default: 11) and `-s` (default: 65).: "The cutoffs for minimum required match length and match score can be changed using the options `-m` (default: 11) and `-s` (default: 65)."
- [readme] verbose output column 4: "4. the length or score of the best match used for classification"
