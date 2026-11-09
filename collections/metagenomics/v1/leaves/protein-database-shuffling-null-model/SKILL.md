---
name: protein-database-shuffling-null-model
description: 'Use when calibrating a minimum match length or score cutoff for a protein-level
  metagenomic classifier: simulate/spurious match length distributions on a composition-preserving
  shuffled database to pick a threshold excluding random matches.'
when_to_use_negative:
- When calibrating nucleotide-level k-mer classifiers such as Kraken (k=31) or Clark
  — this null model addresses protein-level MEM match lengths, not nucleotide k-mer
  thresholds
- When the reads originate from non-protein-coding genomic regions, since protein-level
  classification cannot classify them regardless of cutoff
- When a pre-built or custom database is already the target and no cutoff calibration
  is needed — use kaiju-makedb directly instead of a shuffled null model
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: uShuffle
  role: Shuffles the microbial NR protein database with window length 100 aa to generate
    a composition-preserving null model
- name: Kaiju
  role: Classifies simulated reads in MEM mode against original and shuffled indexes;
    kaiju-mkbwt/kaiju-mkfmi build the indexes
  repo: https://github.com/bioinformatics-centre/kaiju
- name: ART
  role: Simulates HTS reads from whole genomes (including plasmids) used as query
    input
- name: R
  role: Statistical software used to tabulate and plot match-length distributions
- name: ggplot2
  role: Plots the match-length distributions reproducing Supplementary Fig. 5
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:5f7201817d1efffa3191aede5a1d545800204162489de03265618024bb28c6ec
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
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# protein-database-shuffling-null-model

## Summary

Builds a null model of spurious protein matches by shuffling the microbial subset of the NCBI nr protein database with uShuffle (window length 100 aa to preserve local amino acid composition), then comparing Kaiju MEM-mode match-length distributions against the shuffled vs. original index to justify a minimum match length cutoff (m=11).

## When to use

Use when calibrating a minimum match length or score cutoff for a protein-level metagenomic classifier: simulate/spurious match length distributions on a composition-preserving shuffled database to pick a threshold excluding random matches.

## When NOT to use

- When calibrating nucleotide-level k-mer classifiers such as Kraken (k=31) or Clark — this null model addresses protein-level MEM match lengths, not nucleotide k-mer thresholds
- When the reads originate from non-protein-coding genomic regions, since protein-level classification cannot classify them regardless of cutoff
- When a pre-built or custom database is already the target and no cutoff calibration is needed — use kaiju-makedb directly instead of a shuffled null model

## Inputs

- Microbial subset of the NCBI nr protein database (FASTA, uppercase standard 20 amino acids only)
- Simulated metagenomic sequencing reads (FASTQ/FASTA, e.g. from ART)
- NCBI taxonomy files nodes.dmp and names.dmp

## Outputs

- Shuffled-database Kaiju index (.bwt and .fmi files)
- Verbose Kaiju classification output (C/U, read name, taxon ID, best-match length/score in column 4)
- Match-length distributions and a plot (e.g. ggplot2, reproducing Supplementary Fig. 5) supporting the m=11 cutoff

## How to apply

Shuffle the microbial subset of the NCBI nr protein database with uShuffle using a window length of 100 amino acids so local amino acid composition is preserved as a null model. Build Kaiju indexes for both shuffled and original protein sets with kaiju-mkbwt (alphabet ACDEFGHIKLMNPQRSTVWY) followed by kaiju-mkfmi. Classify ART-simulated metagenomic reads against both indexes with Kaiju in MEM mode (-a mem, default minimum match length m=11), then extract best-match lengths from column 4 of the verbose (-v) output and tabulate the length distributions separately for wrong vs. correct classifications in the original-database run. The empirical cutoff is where nearly all shuffled-database matches fall (~95% ≤11 aa) while correct original-database matches rarely do (~2% ≤11 aa); raising m from 11 to 12 loses more precision than the sensitivity gained across all tested data sets.

## Related tools

- **uShuffle** (Shuffles the microbial NR protein database with window length 100 aa to generate a composition-preserving null model)
- **Kaiju** (Classifies simulated reads in MEM mode against original and shuffled indexes; kaiju-mkbwt/kaiju-mkfmi build the indexes) — https://github.com/bioinformatics-centre/kaiju
- **ART** (Simulates HTS reads from whole genomes (including plasmids) used as query input)
- **R** (Statistical software used to tabulate and plot match-length distributions)
- **ggplot2** (Plots the match-length distributions reproducing Supplementary Fig. 5)

## Examples

```
kaiju-mkbwt -n 5 -a ACDEFGHIKLMNPQRSTVWY -o shuffled shuffled.faa; kaiju-mkfmi shuffled; kaiju -t nodes.dmp -f shuffled.fmi -i reads.fastq -a mem -v -o kaiju_shuffled.out
```

## Evaluation signals

- Approximately 95% of matches to the shuffled database are ≤11 amino acids long
- More than 75% of wrong classifications on the original database have matches ≤11 aa, versus only ~2% of correct classifications
- Length distributions reproduce Supplementary Fig. 5 when plotted with ggplot2
- Raising m from 11 to 12 loses more precision than the sensitivity gained, on both genus- and phylum-level in all data sets
- Kaiju index files (.fmi) build cleanly from both original and shuffled FASTA using kaiju-mkbwt with alphabet ACDEFGHIKLMNPQRSTVWY

## Limitations

- The shuffled database preserves local amino acid composition within 100-aa windows, so it models random matches but not real compositional or functional biases
- The m=11 cutoff was validated on ART-simulated reads against the 2014 NCBI genome snapshot and 882-genome benchmark; it may not transfer to other read simulators or database versions
- Protein-level classification cannot classify reads from non-protein-coding genomic regions, so sensitivity losses at higher m include inherently unmatchable reads

## Evidence

- [discussion] we created a shuffled version of the microbial subset of NCBI's NR protein database, using uShuffle25 with a window length of 100 amino acids: "we created a shuffled version of the microbial subset of NCBI's NR protein database, using uShuffle25 with a window length of 100 amino acids"
- [discussion] one can observe that ∼95% of them are ≤11 amino acids long: "one can observe that ∼95% of them are ≤11 amino acids long"
- [readme] kaiju-mkbwt -n 5 -a ACDEFGHIKLMNPQRSTVWY -o proteins proteins.faa: "kaiju-mkbwt -n 5 -a ACDEFGHIKLMNPQRSTVWY -o proteins proteins.faa
kaiju-mkfmi proteins"
- [readme] The cutoffs for minimum required match length and match score can be changed using the options -m (default: 11) and -s (default: 65).: "The cutoffs for minimum required match length and match score can be changed using the options -m (default: 11) and -s (default: 65)."
- [readme] 4. the length or score of the best match used for classification: "4. the length or score of the best match used for classification"
