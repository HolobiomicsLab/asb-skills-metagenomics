---
name: classifier-speed-comparison-analysis
description: Use when you need to quantify and reproduce a classifier's wall-clock
  throughput (reads per minute) on specific metagenome read sets, comparing Kraken
  against Megablast (or MetaPhlAn) with a documented OS-cache priming protocol.
when_to_use_negative:
- Your goal is accuracy or abundance estimation (e.g. MetaPhlAn-style relative abundances),
  not throughput benchmarking — abundance programs classify only a small percentage
  of sequences by design.
- Your read sets are not one of the benchmark metagenomes (HiSeq, MiSeq, or simBA-5),
  since measured throughput and speedup factors are only comparable on the paper's
  datasets.
- You intend to skip or alter the cache-priming step; timing without pre-loading database
  files three times into memory is not comparable to the reported figures.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Kraken
  role: Classifier whose reads-per-minute throughput is measured on the HiSeq, simBA-5
    and MiSeq metagenomes
  repo: github.com/DerrickWood/kraken
- name: Megablast
  role: Fastest BLAST mode used as the speed competitor on the same metagenomes under
    the same cache-priming protocol
- name: MetaPhlAn
  role: Abundance estimation program included in the speed comparison (Kraken reported
    11 times faster)
- name: Mason
  role: Read simulator used to generate the simBA-5 metagenome with its Illumina model
    and 5x mismatch/indel rates
- name: Jellyfish
  role: Multithreaded k-mer counter used to build the Kraken database whose LCA-value
    count spaces are timed during classification
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:5845af7cb6e4ca1edde7fe63e41bab259e4163e701c9feef39505a39a2902ebb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  - Mason
  - Megablast
  - MetaPhlAn
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# classifier-speed-comparison-analysis

## Summary

Benchmarks metagenomic classification throughput (reads per minute) of Kraken against a fastest-BLAST-mode competitor (Megablast) on three defined metagenomes — the HiSeq metagenome, the simulated simBA-5 metagenome, and the MiSeq metagenome — under a cache-priming protocol so speed differences are measured fairly.

## When to use

Use when you need to quantify and reproduce a classifier's wall-clock throughput (reads per minute) on specific metagenome read sets, comparing Kraken against Megablast (or MetaPhlAn) with a documented OS-cache priming protocol.

## When NOT to use

- Your goal is accuracy or abundance estimation (e.g. MetaPhlAn-style relative abundances), not throughput benchmarking — abundance programs classify only a small percentage of sequences by design.
- Your read sets are not one of the benchmark metagenomes (HiSeq, MiSeq, or simBA-5), since measured throughput and speedup factors are only comparable on the paper's datasets.
- You intend to skip or alter the cache-priming step; timing without pre-loading database files three times into memory is not comparable to the reported figures.

## Inputs

- HiSeq metagenome read set (NCBI Sequence Read Archive accession, e.g. SRS019120)
- MiSeq metagenome read set (NCBI Sequence Read Archive accession)
- simBA-5 metagenome generated with Mason from complete bacterial and archaeal RefSeq genomes
- Kraken database built with Jellyfish k-mer counts overwritten by LCA taxon IDs
- Megablast (fastest BLAST mode) installed for competitor timing

## Outputs

- speed_table.csv with measured reads-per-minute values and Kraken-vs-Megablast speedup factors for each of the three metagenomes
- Wall-clock timing records per program per dataset
- Comparison against reported thresholds (1.5M/1.3M/890K rpm) and reported speedup range

## How to apply

Retrieve or generate the three benchmark read sets: the HiSeq and MiSeq metagenomes from their NCBI Sequence Read Archive accessions (SRS019120, SRS014468 and SRS015055), and generate simBA-5 with the Mason read simulator using its Illumina model with mismatch and indel rates multiplied by five (average mismatch rate 2%). Reconstruct the Kraken database by counting k-mers with Jellyfish and overwriting the 4-byte count spaces with LCA taxonomic IDs. Before timing, read each program's database files into memory three times to populate the operating system cache, exactly as the paper's protocol specifies. Run Kraken on each metagenome and record wall-clock time to compute reads per minute, then run Megablast (the fastest BLAST mode) on the same inputs under the same cache-priming protocol and compute the speedup factor per dataset. Validate against the reported values: over 1.5M rpm on HiSeq, over 1.3M rpm on simBA-5, over 890K rpm on MiSeq, with speedup ratios in the reported range; discrepancies beyond these bounds flag the output for expert review.

## Related tools

- **Kraken** (Classifier whose reads-per-minute throughput is measured on the HiSeq, simBA-5 and MiSeq metagenomes) — github.com/DerrickWood/kraken
- **Megablast** (Fastest BLAST mode used as the speed competitor on the same metagenomes under the same cache-priming protocol)
- **MetaPhlAn** (Abundance estimation program included in the speed comparison (Kraken reported 11 times faster))
- **Mason** (Read simulator used to generate the simBA-5 metagenome with its Illumina model and 5x mismatch/indel rates)
- **Jellyfish** (Multithreaded k-mer counter used to build the Kraken database whose LCA-value count spaces are timed during classification)

## Evaluation signals

- Measured Kraken throughput exceeds 1.5 million rpm on the HiSeq metagenome.
- Measured Kraken throughput exceeds 1.3 million rpm on simBA-5.
- Measured Kraken throughput exceeds 890,000 rpm on the MiSeq metagenome.
- Computed Kraken-over-Megablast speedup ratios fall within the reported range (roughly 150–240x on these metagenomes; 909x reported in the abstract for the fastest 100 bp mode).
- Timing was performed only after each program's database files were read into memory three times to populate the OS cache.

## Limitations

- Jellyfish and Mason versions are not specified in the paper, so exact reproduction of the database build and simBA-5 simulation may vary.
- simBA-5 uses elevated error rates (average mismatch rate 2%) via 5x-multiplied Mason defaults; benchmarks on differently simulated or real data are not comparable to the reported figures.
- Paired-end reads were concatenated with 'NNNNN' so each pair is classified as a single unit; classifying mates separately changes reads-per-minute accounting.

## Evidence

- [abstract] Kraken classifies 100 base pair reads at a rate of over 4.1 million reads per minute: "Kraken classifies 100 base pair reads at a rate of over 4.1 million reads per minute"
- [abstract] 909 times faster than Megablast: "909 times faster than Megablast"
- [discussion] Cache priming protocol: [evidence span withheld — anchor status: not_found]
- [discussion] simBA-5 error model: "we multiplied the default mismatch and indel rates by five, resulting in an average mismatch rate of 2%"
- [discussion] simBA-5 provenance: "The simBA-5 metagenome was created by simulating reads from the set of complete bacterial and archaeal genomes in RefSeq"
- [discussion] SRA accessions: "We retrieved the sequences of three accessions (SRS019120, SRS014468 and SRS015055) from the NCBI Sequence Read Archive"
- [readme] README installation pointer: "Please see the [Kraken webpage] or the [Kraken manual] for information on installing and operating Kraken."
