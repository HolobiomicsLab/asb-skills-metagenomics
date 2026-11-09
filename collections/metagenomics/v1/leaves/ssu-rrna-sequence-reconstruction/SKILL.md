---
name: ssu-rrna-sequence-reconstruction
description: Use when input is Illumina paired-end or single-end (meta)genomic FASTQ
  and the goal is near-full-length SSU rRNA gene sequences (not short-read taxonomic
  classification), e.g. to enable species-level 16S/18S assignment or to recover novel
  rRNA variants.
when_to_use_negative:
- When the goal is rapid taxonomic profiling of short reads rather than full-length
  SSU reconstruction — RiboTaxa is not as fast as short-read classifiers (Kraken2,
  Centrifuge)
- When the analysis target is not ribosomal RNA (e.g. whole-genome assembly or mRNA
  expression)
- When read data are amplicon-only 16S PCR products that are already full-length and
  need no reference-targeted assembly
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3174
tools:
- name: BBTools
  role: Adapter removal and quality trimming (ktrim=r, kmer=21, minlength=60, trimq=20,
    qtrim=rl, maxns=1) before reconstruction
  repo: https://github.com/oschakoory/RiboTaxa
- name: SortMeRNA
  role: Filters 16S/18S SSU reads from trimmed metagenomic reads against indexed SILVA
    SSU 138.1; these filtered reads feed EMIRGE
  repo: https://github.com/oschakoory/RiboTaxa
- name: EMIRGE (Emirge_amplicon.py)
  role: Reference-targeted reconstruction of full-length SSU rRNA sequences from SortMeRNA-filtered
    reads (JOIN_THRESHOLD identity = 1, 40 iterations, MIN_COV = 0.3), using usearch
    internally
  repo: https://github.com/oschakoory/RiboTaxa
- name: MetaRib
  role: Reconstructs SSU sequences concurrently from unfiltered high-quality reads
    (minid = 0.96, idfilter = 0.98; 1,000,000 reads subsampled per iteration in benchmarking)
  repo: https://github.com/oschakoory/RiboTaxa
- name: USEARCH
  role: Search/clustering engine used internally by EMIRGE during reconstruction;
    must be installed in $PATH
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:76ce063587e09d30e63dbcec14be70d3860c7384ea784c6e44bee2f0f9e39119
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  repo_url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBTools
  - Centrifuge
  - EMIRGE (Emirge_amplicon.py)
  - Kraken2
  - MetaRib
  - SortMeRNA
  - USEARCH
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ssu-rrna-sequence-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct full-length/near-full-length SSU (16S/18S) rRNA gene sequences from shotgun Illumina metagenomic reads by combining EMIRGE (Emirge_amplicon.py) on SortMeRNA-filtered SSU reads with MetaRib on unfiltered high-quality reads, merging outputs into metagenome_SSU_sequences.fasta. This combined strategy gave greater population characterisation than any single extractor/assembler combination in the RiboTaxa benchmarking.

## When to use

Use when input is Illumina paired-end or single-end (meta)genomic FASTQ and the goal is near-full-length SSU rRNA gene sequences (not short-read taxonomic classification), e.g. to enable species-level 16S/18S assignment or to recover novel rRNA variants.

## When NOT to use

- When the goal is rapid taxonomic profiling of short reads rather than full-length SSU reconstruction — RiboTaxa is not as fast as short-read classifiers (Kraken2, Centrifuge)
- When the analysis target is not ribosomal RNA (e.g. whole-genome assembly or mRNA expression)
- When read data are amplicon-only 16S PCR products that are already full-length and need no reference-targeted assembly

## Inputs

- Illumina paired-end or single-end metagenomic reads in .fastq or fastq.gz (e.g. SRA SRR5381736 mock community)
- Adapter/quality-trimmed high-quality reads produced by BBTools
- Indexed SILVA SSU 138.1 database (SortMeRNA index and Bowtie index)
- USEARCH executable available in $PATH (required internally by EMIRGE)

## Outputs

- *_16S18Sreads.fastq — SSU reads filtered by SortMeRNA
- SortMeRNA .log reporting percentage of SSU reads filtered
- metagenome_SSU_sequences.fasta — combined EMIRGE + MetaRib full-length SSU rRNA reconstructions in the SSU_sequences folder
- Per-reconstruction length distribution and detected variant/microorganism counts (benchmarking context)

## How to apply

First trim adapters and low-quality bases with BBTools (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1) to obtain high-quality reads. Extract 16S/18S reads with SortMeRNA v2.1 against the Bowtie/SortMeRNA-indexed SILVA SSU 138.1 database, producing *_16S18Sreads.fastq plus a .log of the filtered SSU percentage; the article showed SortMeRNA extracts reads that assemble better than BBMap. Feed the SortMeRNA-filtered reads to Emirge_amplicon.py against the Bowtie-indexed SILVA database with JOIN_TRESHOLD/IDENTITY = 1 (merge candidate sequences sharing >= 100% fractional identity), 40 iterations (120 for the gut-community reproduction), MIN_COV = 0.3, using usearch internally. Concurrently reconstruct SSU sequences from the unfiltered trimmed reads with MetaRib (minid = 0.96, idfilter = 0.98, subsampling 1,000,000 reads per iteration). Combine both assembler outputs into a single metagenome_SSU_sequences.fasta in the SSU_sequences output directory, which is consumed downstream by taxonomic classification (QIIME2 sklearn classifier at confidence 0.7) rather than classifying short reads directly, which the article showed yields high false-positive rates.

## Related tools

- **BBTools** (Adapter removal and quality trimming (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1) before reconstruction) — https://github.com/oschakoory/RiboTaxa
- **SortMeRNA** (Filters 16S/18S SSU reads from trimmed metagenomic reads against indexed SILVA SSU 138.1; these filtered reads feed EMIRGE) — https://github.com/oschakoory/RiboTaxa
- **EMIRGE (Emirge_amplicon.py)** (Reference-targeted reconstruction of full-length SSU rRNA sequences from SortMeRNA-filtered reads (JOIN_THRESHOLD identity = 1, 40 iterations, MIN_COV = 0.3), using usearch internally) — https://github.com/oschakoory/RiboTaxa
- **MetaRib** (Reconstructs SSU sequences concurrently from unfiltered high-quality reads (minid = 0.96, idfilter = 0.98; 1,000,000 reads subsampled per iteration in benchmarking)) — https://github.com/oschakoory/RiboTaxa
- **USEARCH** (Search/clustering engine used internally by EMIRGE during reconstruction; must be installed in $PATH)

## Examples

```
conda activate RiboTaxa_py27 && emirge_amplicon.py --help
```

## Evaluation signals

- metagenome_SSU_sequences.fasta exists in the SSU_sequences output folder, is valid FASTA, and contains full-length/nearly full-length 16S/18S sequences from both EMIRGE and MetaRib
- The SortMeRNA .log reports a plausible percentage of SSU reads filtered, and *_16S18Sreads.fastq is non-empty
- On the mock community (SRR5381736), the combined Emirge_amplicon.py + MetaRib workflow followed by 97% clustering yields ~53 reconstructed sequences covering all 20 detectable mock microorganisms (41 expected 16S variants)
- Reconstructed sequences should be correctly assigned downstream (reconstructed sequences were correctly assigned, whereas direct short-read classification gave high false positives)
- Reconstruction is slower than short-read classifiers but stays in the reported minute-scale runtime (RiboTaxa ~18-26 min on the synthetic gut community)

## Limitations

- Chimera detection was not included because no chimeric sequences were detected, but chimeric database sequences could carry over into reconstructions if reads map across the full chimera
- Artificial in silico diversity created during full-length sequence reconstruction cannot be excluded, and occasional small indel errors may occur in reconstructed sequences
- No 16S rRNA gene copy number normalization is applied, so relative abundances are not copy-number corrected
- Near-identical 16S rRNA genes (e.g. some Enterobacteriaceae, Lachnospiraceae) prevent species-level assignment; RiboTaxa stops at genus level for such sequences
- EMIRGE requires USEARCH installed in $PATH, which is not bundled with the pipeline

## Evidence

- [methods] Reconstruction layer splits inputs between EMIRGE and MetaRib: [evidence span withheld — anchor status: not_found]
