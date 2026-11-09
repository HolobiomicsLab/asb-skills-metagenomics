---
name: metagenomic-read-mapping-to-gene-catalog
description: Use when you need to profile the taxonomic and/or functional composition
  of vaginal metagenomic or metatranscriptomic FASTQ samples against the VIRGO gene
  catalog instead of assembling reads de novo.
when_to_use_negative:
- The input is already a feature/abundance table or species profile rather than raw
  FASTQ reads — this skill maps reads, it does not post-process existing profiles.
- 'The sample is a fungal, viral/phage, or non-vaginal community dataset: VIRGO is
  primarily focused on bacteria, with limited inclusion of viral and fungal gene sequences
  (only 10,908 fungal and 15,965 bacteriophage genes).'
- You need nucleotide variant calling within genes or paired-end mapping fidelity
  — VIRGO explicitly does not support paired-end mapping and lacks variant identification
  capability.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VIRGO
  role: Reference database and toolkit whose runMapping.step1.sh/runMapping.step2.sh
    scripts perform read mapping to the non-redundant gene catalog and summarize multi-sample
    statistics
  repo: github.com/ravel-lab/VIRGO
- name: bowtie
  role: Aligner invoked inside runMapping.step1.sh with seed length -l 25 and a reads
    cutoff of 2 for calling a gene hit
  repo: github.com/ravel-lab/VIRGO
- name: BLAST
  role: 'Alternative query route: VIRGO is blastable in both nucleotide and amino
    acid (blastn/blastp, blast 2.2.3+ compatible) for sequence-based lookup against
    the catalog'
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:d560afc58d42101edeaea5001e1a4fb9db3e922675b22bc5296c20d81d87c2e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - VIRGO
  - bowtie
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metagenomic-read-mapping-to-gene-catalog

## Summary

Map metagenomic or metatranscriptomic reads to a body-site-specific non-redundant gene catalog (VIRGO, 0.95 million vaginal non-redundant genes) using bowtie to quantify per-gene read counts, per-species gene counts, and taxonomic abundances. This reference-based approach avoids de novo assembly and enables integration of metagenomic and metatranscriptomic profiles.

## When to use

Use when you need to profile the taxonomic and/or functional composition of vaginal metagenomic or metatranscriptomic FASTQ samples against the VIRGO gene catalog instead of assembling reads de novo.

## When NOT to use

- The input is already a feature/abundance table or species profile rather than raw FASTQ reads — this skill maps reads, it does not post-process existing profiles.
- The sample is a fungal, viral/phage, or non-vaginal community dataset: VIRGO is primarily focused on bacteria, with limited inclusion of viral and fungal gene sequences (only 10,908 fungal and 15,965 bacteriophage genes).
- You need nucleotide variant calling within genes or paired-end mapping fidelity — VIRGO explicitly does not support paired-end mapping and lacks variant identification capability.

## Inputs

- Single-end FASTQ file per sample (paired-end reads merged, since VIRGO does not support paired-end mapping)
- VIRGO non-redundant gene database with Bowtie 1/Bowtie 2 index (0_db) and annotation directory (1_VIRGO)
- Optional list of VIRGO gene IDs of interest (e.g. V1891288) for targeted annotation lookup

## Outputs

- Per-sample sampleName.out file with VIRGO gene ID, mapped read count, and gene length
- summary.Abundance.txt: number of reads per species
- summary.Count.txt: number of genes per species
- summary.Percentage.txt: normalized per-species abundance in percentage (summing to 100)
- summary.geneRichness.txt: number of genes per sample
- Per-gene and per-sample annotation tables (gene.lst.txt, EggNOG.annotation.txt, EC.annotation.txt, Kegg.*.annotation.txt, proteinFamily.annotation.txt, rxn.annotation.txt)

## How to apply

First run VIRGO's test module (runTesting.step1.sh with paired fastq files, then runTesting.step2.sh) to confirm the database and scripts work. Because VIRGO does not support paired-end mapping, merge paired-end reads (alone or with single-end reads) into one single-end FASTQ per sample using seqtk. For each sample run runMapping.step1.sh with -r (single-end fastq), -p (sample prefix), and -d (path to VIRGO); this invokes bowtie with default seed length -l 25 (lower for more hits, higher for stricter mapping) and a default reads cutoff of 2 for calling a gene hit, producing a sampleName.out file (column 1: VIRGO gene ID, column 2: mapped read count, column 3: gene length, sorted by read count). Then run runMapping.step2.sh over the step-1 output directory to generate summary tables: summary.Abundance.txt (reads per species), summary.Count.txt (genes per species), summary.Percentage.txt (normalized species percentages summing to 100), summary.geneRichness.txt (genes per sample), plus per-gene abundance and annotation tables (EggNOG, EC, KEGG ortholog/module/pathway, protein family from CDD, GO, Gene3D, Hamap, Interpro, Pfam, TIGRFAM, etc.).

## Related tools

- **VIRGO** (Reference database and toolkit whose runMapping.step1.sh/runMapping.step2.sh scripts perform read mapping to the non-redundant gene catalog and summarize multi-sample statistics) — github.com/ravel-lab/VIRGO
- **bowtie** (Aligner invoked inside runMapping.step1.sh with seed length -l 25 and a reads cutoff of 2 for calling a gene hit) — github.com/ravel-lab/VIRGO
- **BLAST** (Alternative query route: VIRGO is blastable in both nucleotide and amino acid (blastn/blastp, blast 2.2.3+ compatible) for sequence-based lookup against the catalog) — github.com/ravel-lab/VIRGO

## Examples

```
./runMapping.step1.sh -r sample1.fastq -p samplePrefix -d /full/path/to/VIRGO/ ; ./runMapping.step2.sh -p /path/to/output/of/step1/ -d /path/to/VIRGO/
```

## Evaluation signals

- Module 0 test run produces all expected files (summary.Abundance.txt, summary.Percentage.txt, summary.Count.txt, summary.geneRichness.txt, test.out) confirming VIRGO is functioning before batch processing.
- Each sampleName.out file is sorted by the mapped-read-count column and contains three columns: VIRGO gene ID, number of reads mapped, and gene length.
- summary.Percentage.txt per-species values sum to 100 (normalized percentages).
- Cross-population validation: mapping rates against the complete VIRGO set reproduce >99% for North American, 96% for African, and 88% for Chinese held-out vaginal metagenomes (article reports >95% coverage).
- Per-sample L. crispatus gene counts filtered at >=3 reads per gene reproduce the reported group means of 3262 ± 586 (metagenomes) vs 2064 ± 225 (isolate genomes), a significant 1.6-fold excess at p < 0.001.

## Limitations

- VIRGO does not yet support paired-end mapping; reads must be merged into a single FASTQ, losing pairing information.
- The reference-based approach has acknowledged limitations: this version focuses on gene-level de-redundancy and cannot identify nucleotide variants within a gene.
- The database is primarily bacterial with limited inclusion of viral and fungal gene sequences, so non-bacterial microbes are incompletely profiled.
- Tuning bowtie -l below 25 yields more hits but lower mapping specificity; raising it is stricter but fewer reads map — the reads cutoff (default 2) also changes gene-presence calls.

## Evidence

- [discussion] VIRGO presents a central reference database and analytical framework to enable the efficient and accurate characterization of the microbial gene content of the human vaginal microbiome.: "VIRGO presents a central reference database and analytical framework to enable the efficient and accurate characterization of the microbial gene content of the human vaginal microbiome."
- [intro] we show that VIRGO provides >95% coverage of the human vaginal microbiome, and it is applicable to populations from North America, Africa, and Asia: "we show that VIRGO provides >95% coverage of the human vaginal microbiome, and it is applicable to populations from North America, Africa, and Asia"
- [readme] Please note VIRGO does NOT yet support paired-end mapping, please merge the paired end reads alone or together with single-end reads into one fastq file.: "Please note VIRGO does NOT yet support paired-end mapping, please merge the paired end reads alone or together with single-end reads into one fastq file."
- [readme] in step 1 scripts, the -l is the seed length for read mapping. You can go lower to get more hits or higher to get more strict in mapping quality.: "in step 1 scripts, the -l is the seed length for read mapping. You can go lower to get more hits or higher to get more strict in mapping quality."
- [readme] in step 1 scripts, the cutoff of the number of reads for calling a gene hit.: "in step 1 scripts, the cutoff of the number of reads for calling a gene hit."
