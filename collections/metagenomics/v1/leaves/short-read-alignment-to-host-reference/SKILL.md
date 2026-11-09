---
name: short-read-alignment-to-host-reference
description: Use when microbial short- or long-read (meta)genomic FASTQ (single or
  paired, optionally gzip-compressed) is contaminated with human host sequences that
  must be removed for ethical and legal reasons.
when_to_use_negative:
- Input reads are already non-host or host-depleted (e.g. the bundled tuberculosis
  sample yields reads_removed_proportion of 0.0) and decontamination would add no
  value
- The goal is to keep host reads and discard microbial reads — use `hostile clean
  --invert` instead of the default removal direction
- The goal is to mask human reads with 'N' in place (as NCBI's sra-human-scrubber
  does by default) rather than remove aligned reads from the output FASTQ
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Hostile
  role: orchestrates streaming host-read removal from FASTQ input, wrapping aligner
    and samtools steps
  repo: https://github.com/bede/hostile
- name: Bowtie2
  role: aligns paired short reads to the host index in --sensitive mode with -k 1
- name: Minimap2
  role: aligns long (ONT) reads to the host index in map-ont mode with secondary alignments
    disabled; also used by `hostile mask` to align 150mers for masking
- name: Samtools
  role: counts distinct reads while streaming through the alignment to determine which
    reads to discard
- name: conda
  role: installs Hostile with its non-Python dependencies from bioconda
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:c52a97afe0e9fa4f3217cbce5b5c579e028fac446e495bd8d4663304722a6db4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - Bowtie2
  - Hostile
  - Minimap2
  - Samtools
  - conda
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# short-read-alignment-to-host-reference

## Summary

Streaming decontamination of microbial FASTQ reads by aligning them to a human reference index (Bowtie2 for short reads, Minimap2 for long reads) and discarding aligned reads together with their mates, as implemented in Hostile.

## When to use

Use when microbial short- or long-read (meta)genomic FASTQ (single or paired, optionally gzip-compressed) is contaminated with human host sequences that must be removed for ethical and legal reasons.

## When NOT to use

- Input reads are already non-host or host-depleted (e.g. the bundled tuberculosis sample yields reads_removed_proportion of 0.0) and decontamination would add no value
- The goal is to keep host reads and discard microbial reads — use `hostile clean --invert` instead of the default removal direction
- The goal is to mask human reads with 'N' in place (as NCBI's sra-human-scrubber does by default) rather than remove aligned reads from the output FASTQ

## Inputs

- single-end FASTQ file (optionally gzip-compressed) or stdin
- paired-end FASTQ files (fastq1 and fastq2, optionally gzip-compressed)

## Outputs

- cleaned reads as gzip-compressed FASTQ named <input>.clean.fastq.gz
- JSON summary log with version, aligner, index, fastq1_in_name, reads_in, reads_out, reads_removed and reads_removed_proportion

## How to apply

Install Hostile with its non-Python dependencies (Bowtie2, Minimap2, Samtools, Bedtools) via conda/mamba from bioconda, then run `hostile clean` on fastq[.gz] input or stdin. Align reads against the default `human-t2t-hla` index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51, auto-downloaded on first run) using Bowtie2 in --sensitive mode with -k 1 for paired short reads (auto-selected), or Minimap2 in map-ont mode with secondary alignments disabled for long reads. Stream through the alignment counting distinct reads with Samtools and discard aligned reads together with their mates, keeping data in compressed streams so no intermediate uncompressed FASTQ is created. Optionally replace read headers with incrementing integers (--rename) for privacy, then write cleaned reads as <input>.clean.fastq.gz and a JSON log recording reads_in, reads_out, reads_removed and reads_removed_proportion. For maximum microbial retention, substitute a masked index such as `human-t2t-hla-argos985` or the viral/phage-masked variant via --index.

## Related tools

- **Hostile** (orchestrates streaming host-read removal from FASTQ input, wrapping aligner and samtools steps) — https://github.com/bede/hostile
- **Bowtie2** (aligns paired short reads to the host index in --sensitive mode with -k 1)
- **Minimap2** (aligns long (ONT) reads to the host index in map-ont mode with secondary alignments disabled; also used by `hostile mask` to align 150mers for masking)
- **Samtools** (counts distinct reads while streaming through the alignment to determine which reads to discard)
- **conda** (installs Hostile with its non-Python dependencies from bioconda)

## Examples

```
hostile clean --fastq1 long.fastq.gz  # Creates long.clean.fastq.gz
```

## Evaluation signals

- The JSON log exists, parses, and contains reads_in, reads_out, reads_removed and reads_removed_proportion
- Invariant holds: reads_in == reads_out + reads_removed
- reads_removed_proportion is within [0, 1]
- For an uncontaminated control such as the bundled tuberculosis_1_1.fastq.gz, reads_removed_proportion is 0.0 and reads_out equals reads_in
- Output FASTQ is gzip-compressed and named <input>.clean.fastq.gz, with no intermediate uncompressed FASTQ created

## Limitations

- Short-read decontamination (Bowtie2) requires ~4GB RAM and long-read (Minimap2) ~13GB, which may constrain laptops with less memory
- A human reference genome/index (e.g. human-t2t-hla) is downloaded on first run unless another index is specified; earlier versions did not verify downloaded index checksums (fixed in issue #20)
- Earlier versions could corrupt empty fastq.gz files when no reads remain after decontamination (fixed in issue #24)
- Unmasked default index removes more microbial reads; masked indexes trade ≤0.001% human-read removal for ≥99.997% bacterial retention, so index choice depends on whether viral/bacterial retention is a priority

## Evidence

- [intro] Hostile removes host sequences from short and long read (meta)genomes, consuming single or paired FASTQ from files or stdin.: "Hostile removes host sequences from short and long read (meta)genomes, consuming single or paired FASTQ from files or stdin."
- [methods] Remove reads aligning to an index from fastq[.gz] input files or stdin.: "Remove reads aligning to an index from fastq[.gz] input files or stdin."
- [methods] Installation with conda/mamba or Docker is recommended due to non-Python dependencies (Bowtie2, Minimap2, Samtools and Bedtools): "Installation with conda/mamba or Docker is recommended due to non-Python dependencies (Bowtie2, Minimap2, Samtools and Bedtools)"
- [methods] The default index `human-t2t-hla` comprises [T2T-CHM13v2.0](https://www.ncbi.nlm.nih.gov/assembly/11828891) and [IPD-IMGT/HLA](https://www.ebi.ac.uk/ipd/imgt/hla/) v3.51: "The default index `human-t2t-hla` comprises"
- [readme] Read headers can be replaced with integers (using `--rename`) for privacy and smaller FASTQs.: "Read headers can be replaced with integers (using `--rename`) for privacy and smaller FASTQs."
- [readme] In benchmarks, bacterial Illumina reads were decontaminated at 32Mbp/s (210k reads/sec) and bacterial ONT reads at 22Mbp/s, using 8 alignment threads.: "In benchmarks, bacterial Illumina reads were decontaminated at 32Mbp/s (210k reads/sec) and bacterial ONT reads at 22Mbp/s, using 8 alignment threads."
