---
name: simulated-read-classification-benchmarking
description: 'Use when evaluating a read-level metagenomic classifier (e.g., Kaiju
  vs Kraken/Clark) against a ground truth: simulate reads with ART, classify them,
  and measure sensitivity and precision.'
when_to_use_negative:
- Evaluating strain-level sensitivity when the sequenced organism IS in the reference
  database — protein-level classifiers like Kaiju are less sensitive than nucleotide-level
  classifiers in that setting
- Classifying reads from non-protein-coding genomic regions, which protein-level classifiers
  cannot assign
- Benchmarking genus-level recovery for genera with no available reference genomes
  — such classification is impossible by design
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: Classifier under test; run in Greedy (default, -e 3, -s 65, -E 0.01) and MEM
    (-a mem) modes on the same FASTQ and .fmi index
  repo: https://github.com/bioinformatics-centre/kaiju
- name: ART
  role: Simulates HTS reads from whole genomes (including plasmids) to create ground-truth
    benchmark read sets
- name: Kraken
  role: k-mer-based comparator classifier (v0.10.4b, k=31) run in default mode for
    sensitivity/precision comparison
- name: Clark
  role: k-mer-based comparator classifier (v1.1.3) run in default mode for sensitivity/precision
    comparison
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:afa28b8094e4b5137eaf0f1a022d0d012bc240cc1281ed1c410b563e1d26c9a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Clark
  - Kaiju
  - Kraken
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# simulated-read-classification-benchmarking

## Summary

Benchmark a taxonomic classifier by simulating HTS reads with ART from reference genomes (optionally excluding the source genome from the database), classifying them, and computing sensitivity and precision of the assignments.

## When to use

Use when evaluating a read-level metagenomic classifier (e.g., Kaiju vs Kraken/Clark) against a ground truth: simulate reads with ART, classify them, and measure sensitivity and precision.

## When NOT to use

- Evaluating strain-level sensitivity when the sequenced organism IS in the reference database — protein-level classifiers like Kaiju are less sensitive than nucleotide-level classifiers in that setting
- Classifying reads from non-protein-coding genomic regions, which protein-level classifiers cannot assign
- Benchmarking genus-level recovery for genera with no available reference genomes — such classification is impossible by design

## Inputs

- Reference genomes with annotated proteins (e.g., NCBI RefSeq Archaea, Bacteria, viruses; plasmids included)
- NCBI taxonomy files nodes.dmp and names.dmp
- Protein reference database FASTA with numeric NCBI taxon ID headers, converted to a Kaiju FM-index (kaiju-mkbwt + kaiju-mkfmi)
- Simulated sequencing reads in FASTQ format produced by ART

## Outputs

- Kaiju-style three-column classification output (C/U flag, read name, NCBI taxon identifier), optionally verbose with match score/length
- Per-mode classification rates (sensitivity, precision) and cross-mode agreement, e.g., a comparison table like greedy_vs_mem_comparison.tsv

## How to apply

Simulate reads with ART from whole genomes (including plasmids) and classify them with the classifier under test against a reference database; for the genome-exclusion design, build the database without the source genome so distant-homology sensitivity can be measured. Run each classifier mode or tool on the identical FASTQ inputs and identical index (e.g., Kaiju Greedy mode with defaults: -e 3 mismatches, -m 11 min length, -s 65 min score, -E 0.01 E-value, versus -a mem), emitting the three-column C/U output with NCBI taxon IDs. Because simulated reads carry known origins, classify each read as a true/false positive by checking whether the assigned taxon is the source taxon or an ancestor (e.g., correct LCA assignment) in nodes.dmp, then compute per-mode sensitivity (fraction of reads classified) and precision. Compare classified read names and taxon IDs between runs (e.g., writing a comparison table) and confirm the expectation that the more sensitive mode (Greedy) classifies at least as many reads at comparable precision.

## Related tools

- **Kaiju** (Classifier under test; run in Greedy (default, -e 3, -s 65, -E 0.01) and MEM (-a mem) modes on the same FASTQ and .fmi index) — https://github.com/bioinformatics-centre/kaiju
- **ART** (Simulates HTS reads from whole genomes (including plasmids) to create ground-truth benchmark read sets)
- **Kraken** (k-mer-based comparator classifier (v0.10.4b, k=31) run in default mode for sensitivity/precision comparison)
- **Clark** (k-mer-based comparator classifier (v1.1.3) run in default mode for sensitivity/precision comparison)

## Examples

```
kaiju -z 25 -t nodes.dmp -f kaiju_db.fmi -i simulated_reads.fastq -a mem -o kaiju_simulated.out
```

## Evaluation signals

- Reads simulated from a genome present in the database are flagged C and assigned the correct taxon or a correct LCA ancestor; reads without adequate matches are flagged U
- In the genome-exclusion benchmark, the tested mode achieves higher sensitivity (more reads classified) with little or no loss of precision versus Kraken (k=31) and Clark baselines
- All emitted taxon identifiers resolve to valid nodes in nodes.dmp (no assignments outside the reference taxonomy)
- Greedy vs MEM comparison shows Greedy classifies at least as many reads at comparable precision, matching the paper's reported higher sensitivity of Greedy mode
- Output is one line per read (or read pair) in the three-column tab-separated C/U format with no dropped reads

## Limitations

- Allowing substitutions in the backwards search causes an exponential runtime increase with the number of allowed mismatches
- The greedy heuristic restricts substitutions to match ends rather than all query positions, so inexact-match sensitivity is bounded
- Kaiju can process millions of reads per minute, but classification speed depends on read length and the number of allowed mismatches
- Results depend on database choice: underrepresented genera yield lower sensitivity, and protein-level classification misses non-coding regions

## Evidence

- [discussion] Genome-exclusion benchmark design: "For each of the 882 genomes, we simulated five sets of HTS reads and created a reference database not containing this genome that is then used to classify the simulated reads."
- [discussion] ART simulation scope: "Reads were simulated from the whole genome (including plasmids) using ART27."
- [discussion] Benchmark outcome: "Our benchmark on 882 genomes and five types of simulated reads shows that Kaiju consistently achieves a much higher sensitivity with only little loss of precision compared with Kraken and Clark"
- [readme] Run modes and cutoffs: "The default run mode is **Greedy** with three allowed mismatches. The number of allowed mismatches can be changed using option `-e`."
- [readme] Output schema: "Kaiju will print one line for each read or read pair.
The default output format contains three columns separated by tabs."
