---
name: intraspecies-gene-content-comparison
description: Use when metagenomic reads (single-end merged FASTQ) can be mapped to
  a reference gene catalog and you want to test whether within-community intraspecies
  strain co-occurrence inflates per-sample gene counts beyond isolate pangenomes.
when_to_use_negative:
- The goal is strain-level variant or SNP calling — VIRGO does not yet identify nucleotide
  variants within genes, so gene-count excess is only an indirect strain co-occurrence
  signal
- The species of interest is poorly represented in VIRGO's isolate-backed reference,
  since gaps may reflect limited isolate genomes or systematic bias in growth and
  recovery of vaginal species rather than true co-occurring strains
- Input data are 16S rRNA amplicon profiles, which cannot resolve accessory-gene content
  differences between strains
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: VIRGO
  role: Reference gene catalog and runMapping/runTesting scripts used to map reads,
    tabulate per-species gene counts, and detect metagenomic intraspecies gene excess
  repo: github.com/ravel-lab/VIRGO
- name: bowtie
  role: Short-read mapper invoked inside runMapping.step1.sh with seed length -l 25
    and reads cutoff of 2 for calling a gene hit
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:9e35a03e95960e4afecc8751ce2e238f0c28439d663791c4e303c8ad30ff18ab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - VIRGO
  - bowtie
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# intraspecies-gene-content-comparison

## Summary

Quantify whether a species' gene content recovered from vaginal metagenomes exceeds that of its isolate genomes, using VIRGO mapping to detect co-occurring strains (e.g., L. crispatus metagenomes show 3262 ± 586 genes vs 2064 ± 225 in isolates, a 1.6-fold excess, p < 0.001).

## When to use

Use when metagenomic reads (single-end merged FASTQ) can be mapped to a reference gene catalog and you want to test whether within-community intraspecies strain co-occurrence inflates per-sample gene counts beyond isolate pangenomes.

## When NOT to use

- The goal is strain-level variant or SNP calling — VIRGO does not yet identify nucleotide variants within genes, so gene-count excess is only an indirect strain co-occurrence signal
- The species of interest is poorly represented in VIRGO's isolate-backed reference, since gaps may reflect limited isolate genomes or systematic bias in growth and recovery of vaginal species rather than true co-occurring strains
- Input data are 16S rRNA amplicon profiles, which cannot resolve accessory-gene content differences between strains

## Inputs

- Merged single-end FASTQ files from 1507 vaginal metagenomes (paired-end reads combined, since VIRGO does not support paired-end mapping)
- FASTQ read sets for L. crispatus isolate genomes
- VIRGO non-redundant gene catalog (0.95 million genes) and its Bowtie 1 database from /path/to/VIRGO/0_db

## Outputs

- Per-sample .out files (VIRGO gene ID, mapped read count, gene length) from runMapping.step1.sh
- summary.Count.txt — number of genes per species per sample for metagenomes and isolate genomes
- summary.geneRichness.txt — number of genes per sample
- Group means and Welch-type test statistic showing 3262 ± 586 vs 2064 ± 225 genes at p < 0.001

## How to apply

First validate the VIRGO installation with the test module (runTesting.step1.sh and runTesting.step2.sh), then merge paired-end reads into single-end FASTQ because VIRGO does not support paired-end mapping. Run runMapping.step1.sh (bowtie with seed length -l 25 and a reads cutoff of 2 for calling a gene hit) on each of the 1507 vaginal metagenomes and each L. crispatus isolate genome read set, producing per-sample .out files of VIRGO gene IDs, mapped read counts, and gene lengths. Run runMapping.step2.sh to build summary tables, especially summary.Count.txt (genes per species per sample) and summary.geneRichness.txt. Filter mapped genes for presence at >=3 reads per gene and call a species' gene content present in a community only when it reaches >=80% of the average gene count across that species' isolate genomes. Finally compare mean per-sample gene counts between metagenome-derived and isolate-genome groups (Welch-type test) to confirm the 1.6-fold excess.

## Related tools

- **VIRGO** (Reference gene catalog and runMapping/runTesting scripts used to map reads, tabulate per-species gene counts, and detect metagenomic intraspecies gene excess) — github.com/ravel-lab/VIRGO
- **bowtie** (Short-read mapper invoked inside runMapping.step1.sh with seed length -l 25 and reads cutoff of 2 for calling a gene hit) — github.com/ravel-lab/VIRGO

## Examples

```
./runMapping.step1.sh -r sample1.fastq -p samplePrefix -d /full/path/to/VIRGO/ && ./runMapping.step2.sh -p /path/to/output/of/step1/ -d /path/to/VIRGO/
```

## Evaluation signals

- summary.Count.txt exists and contains per-sample gene-count rows for both metagenome and isolate-genome groups
- Mean L. crispatus gene counts reproduce 3262 ± 586 (metagenomes) and 2064 ± 225 (isolate genomes)
- Welch-type test on per-sample counts yields a significant difference at p < 0.001 (about 1.6-fold excess)
- Each per-sample .out file has three columns (VIRGO gene ID, reads mapped, gene length) sorted by the reads-mapped column
- Test run with runTesting.step1.sh/step2.sh produces the expected outputs including summary.Count.txt, summary.geneRichness.txt, and test.out

## Limitations

- VIRGO does not support paired-end mapping; reads must be merged into a single FASTQ, which loses mate-pair information
- The reference-based approach is limited: the database is primarily bacterial with limited viral/fungal inclusion, and observed metagenomic subspecies contain non-redundant genes not seen in isolate genomes
- Excess gene counts could reflect limited isolate genome availability or systematic bias in growth and recovery of vaginal species, requiring targeted isolation to disambiguate

## Evidence

- [results] L. crispatus genes identified in vaginal metagenomes contained on average 1.6 times more genes (3262 ± 586) than L. crispatus isolate genomes (2064 ± 225): [evidence span withheld — anchor status: not_found]
- [readme] reads cutoff default: "in step 1 scripts, the cutoff of the number of reads for calling a gene hit."
- [readme] bowtie seed length: "the -l is the seed length for read mapping. You can go lower to get more hits or higher to get more strict in mapping quality."
- [readme] paired-end not supported: "Please note VIRGO does NOT yet support paired-end mapping, please merge the paired end reads alone or together with single-end reads into one fastq file."
- [readme] summary.Count.txt output description: "summary.Count.txt: the number of genes per species"
- [discussion] non-redundant genes not in isolates: "the observed metagenomic subspecies contain non-redundant genes, which have not been observed in isolate genome sequences for that species"
