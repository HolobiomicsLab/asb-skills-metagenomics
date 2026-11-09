---
name: metagenomic-read-quality-control
description: Use when raw vaginal metagenomic or metatranscriptomic fastq reads must
  be cleaned (human/rRNA removal) and mapped to a reference gene catalog such as VIRGO
  before taxonomic or functional profiling.
when_to_use_negative:
- Input is already a gene-count or abundance table from a prior VIRGO mapping run
  — proceed directly to downstream summarization instead of re-QC
- The goal is de novo assembly-based CDS prediction (as in building the catalog itself)
  rather than reference-based read mapping to an existing gene catalog
- VIRGO's bacterial-focused catalog is inappropriate, e.g. the target community is
  dominated by viral or fungal genes, which are only sparsely included (see 2_fungal_phage
  annotations)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: VIRGO
  role: Reference non-redundant gene database and toolkit that filtered reads are
    mapped against using runMapping.step1.sh / step2.sh scripts
  repo: github.com/ravel-lab/VIRGO
- name: bowtie
  role: Short-read mapper used in step 1 scripts; the -l seed length parameter (default
    25) controls mapping stringency
- name: BLAST
  role: VIRGO databases are blastable in both nucleotide and amino acid (blastn/blastp)
    as an alternative query route; blast 2.2.3+ compatible
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:e4cdafda593e0e49a8cfcdb88d143927ba3ee621ee8eaacda3aff9986fd297b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - VIRGO
  - bowtie
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metagenomic-read-quality-control

## Summary

Quality-control and filter metagenomic/metatranscriptomic fastq reads before gene-catalog analysis, including removing host-derived reads and rRNA, then mapping filtered reads to the VIRGO non-redundant gene database with explicit read-count and seed-length cutoffs.

## When to use

Use when raw vaginal metagenomic or metatranscriptomic fastq reads must be cleaned (human/rRNA removal) and mapped to a reference gene catalog such as VIRGO before taxonomic or functional profiling.

## When NOT to use

- Input is already a gene-count or abundance table from a prior VIRGO mapping run — proceed directly to downstream summarization instead of re-QC
- The goal is de novo assembly-based CDS prediction (as in building the catalog itself) rather than reference-based read mapping to an existing gene catalog
- VIRGO's bacterial-focused catalog is inappropriate, e.g. the target community is dominated by viral or fungal genes, which are only sparsely included (see 2_fungal_phage annotations)

## Inputs

- single-end fastq reads from vaginal metagenomes or metatranscriptomes (paired-end reads merged into one fastq)
- quality-filtered fastq reads with human-derived reads and rRNA sequences removed
- local VIRGO directory containing 0_db (Bowtie 1/2, BLASTN, BLASTP databases) and 1_VIRGO annotation files

## Outputs

- sampleName.out file per sample: VIRGO gene ID, number of reads mapped, gene length, sorted by mapped-read count
- summary.Abundance.txt, summary.Count.txt, summary.Percentage.txt, summary.geneRichness.txt across samples
- annotation tables per sample (EggNOG.annotation.txt, EC.annotation.txt, geneProduct.txt, Kegg.*.annotation.txt, proteinFamily.annotation.txt, rxn.annotation.txt)

## How to apply

Preprocess each of the vaginal metagenomic read sets by removing human-derived reads and rRNA sequences, applying per-sample QC to the remaining fastq reads before any assembly or mapping. Merge paired-end reads into a single fastq file, since VIRGO does not support paired-end mapping, and pass single-end reads to runMapping.step1.sh with the VIRGO directory path. Tune the step-1 script parameters: the 'reads' cutoff (default 2) sets the minimum number of reads required to call a gene hit, and 'bowtie -l' (default 25) sets the seed length for read mapping — lower it to recover more hits or raise it for stricter mapping quality. Run runMapping.step2.sh across all samples to summarize reads per species, genes per species, normalized percentage abundance, and gene richness into the summary.*.txt tables. Judge success by whether per-sample outputs (sampleName.out sorted by mapped-read counts, summary.Abundance.txt, summary.Percentage.txt, summary.geneRichness.txt) are produced consistently across all samples.

## Related tools

- **VIRGO** (Reference non-redundant gene database and toolkit that filtered reads are mapped against using runMapping.step1.sh / step2.sh scripts) — github.com/ravel-lab/VIRGO
- **bowtie** (Short-read mapper used in step 1 scripts; the -l seed length parameter (default 25) controls mapping stringency)
- **BLAST** (VIRGO databases are blastable in both nucleotide and amino acid (blastn/blastp) as an alternative query route; blast 2.2.3+ compatible)

## Examples

```
./runMapping.step1.sh -r sample1.fastq -p samplePrefix -d /full/path/to/VIRGO/; ./runMapping.step2.sh -p /path/to/output/of/step1/ -d /path/to/VIRGO/
```

## Evaluation signals

- Each sample produces a sampleName.out file whose first column is a VIRGO gene ID, second column mapped-read count, third column gene length, sorted by mapped reads (e.g. V1593031 2425 3663)
- The reads-cutoff parameter (default 2) and bowtie -l seed length (default 25) are set explicitly and documented for the run
- runMapping.step2.sh yields the full expected output set: summary.Abundance.txt, summary.Count.txt, summary.Percentage.txt, summary.geneRichness.txt, gene.lst.txt, and the per-sample annotation files
- summary.Percentage.txt values sum to 100 (normalized abundance per species)
- Test-run mode reproduces the expected _test_run output files (summary.Abundance.txt, summary.Percentage.txt, summary.Count.txt, summary.geneRichness.txt, test.1.reads2ref, test.annotation.txt)

## Limitations

- VIRGO does not yet support paired-end mapping; paired-end reads must be merged alone or together with single-end reads into one fastq file
- The reference-based approach of VIRGO is a acknowledged limitation; this version focuses on gene-level de-redundancy of the vaginal microbiome
- The database is primarily focused on bacteria with limited inclusion of viral and fungal gene sequences (10,908 fungal and 15,965 bacteriophage genes are annotated separately)
- Nucleotide-variant identification within genes is not yet included; planned for a future version

## Evidence

- [methods] Preprocess the 264 vaginal metagenomic read sets by removing human-derived reads and rRNA sequences, with per-sample QC on the remaining fastq reads.: [evidence span withheld — anchor status: not_found]
- [readme] read cutoff for calling a gene hit (default 2) and bowtie seed length (default 25) are the tunable QC/mapping thresholds: "in step 1 scripts, the cutoff of the number of reads for calling a gene hit."
- [readme] bowtie seed length governs mapping stringency: "the -l is the seed length for read mapping. You can go lower to get more hits or higher to get more strict in mapping quality."
- [readme] VIRGO requires single-end input; paired reads must be merged: "Please note VIRGO does NOT yet support paired-end mapping, please merge the paired end reads alone or together with single-end reads into one fastq file."
- [abstract] VIRGO covers the vaginal microbiome and supports taxonomic/functional profiling of metagenomic and metatranscriptomic data: "VIRGO offers a convenient reference database and toolkit that will facilitate a more in-depth understanding of the role of vaginal microorganisms"
