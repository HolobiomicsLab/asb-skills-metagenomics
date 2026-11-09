---
name: emirge-iterative-candidate-merging
description: Use when adapter-trimmed Illumina (meta)genomic reads must be turned
  into full-length/nearly full-length SSU rRNA gene sequences for species-level taxonomy,
  and a reference-targeted, iterative assembler with candidate merging is wanted.
when_to_use_negative:
- Input is already assembled full-length 16S/18S sequences or a feature table — reconstruction
  would be redundant
- Direct classification of the short Illumina reads is the goal (e.g. Kraken2/Centrifuge);
  the paper notes read-level classification yields a high rate of false positives,
  but RiboTaxa is also not as fast as short-read classifiers
- Data are amplicon reads rather than shotgun (meta)genomic Illumina data — RiboTaxa
  takes metagenomics Illumina sequences as input
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0092
tools:
- name: EMIRGE
  role: Reference-targeted iterative assembler (Emirge_amplicon.py) reconstructing
    SSU rRNA genes from SortMeRNA-filtered reads with JOIN_TRESHOLD/IDENTITY = 1 and
    120 iterations
  repo: https://github.com/oschakoory/RiboTaxa
- name: SortMeRNA
  role: rRNA-specialized tool used to extract the 16S/18S SSU reads that are fed to
    EMIRGE (outperformed BBMap for extracting assemblable rRNA reads)
- name: MetaRib
  role: Parallel SSU reconstructor run on unfiltered high-quality reads (minid = 0.96,
    idfilter = 0.98); outputs are combined with EMIRGE's
  repo: https://github.com/oschakoory/RiboTaxa
- name: USEARCH
  role: Internal dependency invoked by EMIRGE during full-length rRNA sequence reconstruction;
    must be installed in $PATH
- name: BBTools
  role: Upstream adapter/quality trimming (ktrim=r, kmer=21, minlength=60, trimq=20,
    qtrim=rl, maxns=1) producing the reads that enter the SSU read-extraction step
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:0d9e4749aeb325b1fcd9fbc2c8e716f00dc424ad7cae616c2d285f847d2dc665
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
  - EMIRGE
  - MetaRib
  - SortMeRNA
  - USEARCH
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# emirge-iterative-candidate-merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct full-length SSU rRNA (16S/18S) gene sequences from SortMeRNA-filtered Illumina metagenomics reads using EMIRGE's Emirge_amplicon.py, which iteratively merges candidate reference sequences via a JOIN_TRESHOLD identity criterion, and combine the result with MetaRib reconstructions.

## When to use

Use when adapter-trimmed Illumina (meta)genomic reads must be turned into full-length/nearly full-length SSU rRNA gene sequences for species-level taxonomy, and a reference-targeted, iterative assembler with candidate merging is wanted.

## When NOT to use

- Input is already assembled full-length 16S/18S sequences or a feature table — reconstruction would be redundant
- Direct classification of the short Illumina reads is the goal (e.g. Kraken2/Centrifuge); the paper notes read-level classification yields a high rate of false positives, but RiboTaxa is also not as fast as short-read classifiers
- Data are amplicon reads rather than shotgun (meta)genomic Illumina data — RiboTaxa takes metagenomics Illumina sequences as input

## Inputs

- Adapter-trimmed high-quality Illumina single-end or paired-end reads (.fastq or .fastq.gz) from BBTOOLS QC
- SortMeRNA-filtered SSU read files (*_16S18Sreads.fastq) extracted against the indexed SILVA SSU 138.1 database
- Bowtie-indexed SILVA SSU 138.1 reference database for EMIRGE

## Outputs

- EMIRGE-reconstructed SSU rRNA candidate sequences from the SortMeRNA-filtered reads
- metagenome_SSU_sequences.fasta in the SSU_sequences output directory combining EMIRGE and MetaRib reconstructions
- SortMeRNA .log reporting the percentage of SSU reads filtered

## How to apply

Start from adapter-trimmed high-quality reads produced by BBTOOLS QC (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1), then extract 16S/18S reads with SortMeRNA against the pre-indexed SILVA SSU 138.1 database, yielding *_16S18Sreads.fastq. Feed those SSU reads to Emirge_amplicon.py (EMIRGE) against the Bowtie-indexed SILVA database, setting the EMIRGE JOIN_TRESHOLD/IDENTITY to 1 so two candidate sequences are merged only when they share >= this fractional identity over their bases with mapped reads, and running 120 iterations; EMIRGE uses USEARCH internally, so verify usearch is on $PATH. Run MetaRib in parallel on the unfiltered high-quality reads with minid = 0.96 and idfilter = 0.98, because combining the two tools for SSU reconstruction gave greater population characterisation in the RiboTaxa paper. Merge the EMIRGE and MetaRib outputs into a single metagenome_SSU_sequences.fasta in the SSU_sequences output directory, which is the validated deliverable of this reconstruction stage.

## Related tools

- **EMIRGE** (Reference-targeted iterative assembler (Emirge_amplicon.py) reconstructing SSU rRNA genes from SortMeRNA-filtered reads with JOIN_TRESHOLD/IDENTITY = 1 and 120 iterations) — https://github.com/oschakoory/RiboTaxa
- **SortMeRNA** (rRNA-specialized tool used to extract the 16S/18S SSU reads that are fed to EMIRGE (outperformed BBMap for extracting assemblable rRNA reads))
- **MetaRib** (Parallel SSU reconstructor run on unfiltered high-quality reads (minid = 0.96, idfilter = 0.98); outputs are combined with EMIRGE's) — https://github.com/oschakoory/RiboTaxa
- **USEARCH** (Internal dependency invoked by EMIRGE during full-length rRNA sequence reconstruction; must be installed in $PATH)
- **BBTools** (Upstream adapter/quality trimming (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1) producing the reads that enter the SSU read-extraction step)

## Examples

```
conda activate RiboTaxa_py27 && emirge_amplicon.py --help
```

## Evaluation signals

- metagenome_SSU_sequences.fasta exists, is valid FASTA, and contains the final full-length/nearly full-length rRNA 16S/18S gene sequences reconstructed by both EMIRGE and MetaRib
- The SortMeRNA .log reports the percentage of SSU reads filtered from the trimmed (meta)genomics sequence files
- Candidate sequences are only merged when they share >= JOIN_TRESHOLD fractional identity (set to 1) over their bases with mapped reads, per EMIRGE's parameter semantics
- usearch is resolvable in $PATH (usearch --version succeeds) since EMIRGE requires it internally
- The reconstruction runs inside the RiboTaxa_py27 conda environment where emirge_amplicon.py is available

## Limitations

- Chimera detection was not included because no chimeric sequences were detected during 16S rRNA gene reconstruction, but chimeric database sequences could carry over into SSU reconstructions if reads map across the full length of the chimera
- A part of the reconstructed diversity may be artificial in-silico diversity created during full-length sequence reconstruction, and occasional small indel errors can occur in reconstructed sequences
- RiboTaxa (and this EMIRGE/MetaRib reconstruction stage) is slower than short-read classifiers such as Kraken2 and Centrifuge

## Evidence
