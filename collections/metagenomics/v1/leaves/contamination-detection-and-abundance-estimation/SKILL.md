---
name: contamination-detection-and-abundance-estimation
description: Use when raw nanopore signal reads (FAST5/POD5/SLOW5/BLOW5) from a sample
  with possible mixed organisms must be assigned to source genomes and per-organism
  relative abundances or contamination levels must be quantified from the mapped reads.
when_to_use_negative:
- Input is already basecalled FASTQ/FASTA — use a sequence-based mapper like minimap2
  directly instead of raw-signal mapping
- Data is r10.4.1 chemistry — UNCALLED only supports legacy r9.4.1 data, so comparisons
  on newer chemistries are out of scope
- The task is de novo assembly without a reference genome — use Rawsamble's all-vs-all
  overlapping instead of reference-based mapping
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: RawHash
  role: Primary signal mapper used to map raw nanopore signals to multi-genome references,
    whose mappings feed contamination detection and abundance estimation
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash2
  role: More sensitive and faster variant of RawHash with improved quantization, used
    to build the reference index for mapping
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Baseline signal mapper compared against RawHash; its PAF template and pafstats
    utility define the output format and scoring
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Baseline streaming signal mapper compared against RawHash on the contamination
    dataset
  repo: https://github.com/haowenz/sigmap
- name: Minimap2
  role: Maps basecalled reads to generate ground-truth alignments for scoring contamination/abundance
    accuracy
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:ed4b1c541f49ec0f34b2c5e400a690281b5a105817033e30920d802c5baebfe0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - Minimap2
  - RawHash
  - RawHash2
  - Sigmap
  - UNCALLED
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# contamination-detection-and-abundance-estimation

## Summary

A downstream analysis pattern for raw nanopore signal mapping in which reads mapped by signal-based mappers (RawHash, RawHash2, UNCALLED, Sigmap) to a multi-genome reference are aggregated to detect contaminant organisms and estimate relative abundances, as evaluated in the RawHash paper's contamination analysis and relative abundance estimation applications.

## When to use

Use when raw nanopore signal reads (FAST5/POD5/SLOW5/BLOW5) from a sample with possible mixed organisms must be assigned to source genomes and per-organism relative abundances or contamination levels must be quantified from the mapped reads.

## When NOT to use

- Input is already basecalled FASTQ/FASTA — use a sequence-based mapper like minimap2 directly instead of raw-signal mapping
- Data is r10.4.1 chemistry — UNCALLED only supports legacy r9.4.1 data, so comparisons on newer chemistries are out of scope
- The task is de novo assembly without a reference genome — use Rawsamble's all-vs-all overlapping instead of reference-based mapping

## Inputs

- Raw nanopore signal files (FAST5, POD5, SLOW5, or BLOW5) from MinION runs, e.g., the contamination MinION FAST5 files
- Reference genome sequences in FASTA format (one or multiple candidate organisms)
- ONT k-mer pore model (e.g., extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model for r9.4.1 chemistry)
- Basecalled-read alignments from minimap2 as ground truth

## Outputs

- PAF files of mapped raw signals per tool (UNCALLED-style PAF template)
- Contamination detection calls per organism
- Relative abundance estimates per organism
- Aggregated accuracy_table.csv with per-tool, per-dataset precision/recall/F1 and error rates

## How to apply

Build signal-mapper indexes for each candidate reference genome with the organism's matching pore model (e.g., rawhash2 with extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model for r9.4 data). Map the raw signals from the contamination MinION FAST5 dataset with RawHash, UNCALLED, and Sigmap using default parameters, emitting mappings in the UNCALLED-style PAF template. For each organism, count or sum mapped read mass from the PAF output and derive contamination detection calls and relative abundance estimates; validate the abundance/contamination calls against minimap2 alignments of basecalled reads used as ground truth. Correctness is judged by whether the reconstructed accuracy_table reproduces the paper's ordering — RawHash achieving the best precision, recall, and F1 on the large-genome and contamination analyses while staying on par for smaller genomes.

## Related tools

- **RawHash** (Primary signal mapper used to map raw nanopore signals to multi-genome references, whose mappings feed contamination detection and abundance estimation) — https://github.com/CMU-SAFARI/RawHash
- **RawHash2** (More sensitive and faster variant of RawHash with improved quantization, used to build the reference index for mapping) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Baseline signal mapper compared against RawHash; its PAF template and pafstats utility define the output format and scoring) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Baseline streaming signal mapper compared against RawHash on the contamination dataset) — https://github.com/haowenz/sigmap
- **Minimap2** (Maps basecalled reads to generate ground-truth alignments for scoring contamination/abundance accuracy) — https://github.com/lh3/minimap2

## Examples

```
rawhash2 -d ref.ind -p extern/kmer_models/legacy/legacy_r9.4_180mv_450bps_6mer/template_median68pA.model -t 32 ref.fasta
```

## Evaluation signals

- accuracy_table.csv contains precision, recall, F1, and error-rate values for all three tools (RawHash, UNCALLED, Sigmap) on the contamination dataset
- Reconstructed relative abundances reproduce the paper's qualitative ordering: RawHash significantly better accuracy than UNCALLED and Sigmap for large genomes, on par for smaller genomes
- Every tool's PAF output conforms to the UNCALLED-style PAF template before scoring
- pafstats scoring against minimap2 ground truth completes without unmapped/unmatched read-name errors, confirming consistent read identifiers across tools
- Contamination calls agree with the known organism composition of the MinION contamination FAST5 dataset

## Limitations

- Raw signal mapping accuracy depends on the availability of the correct ONT pore model for the chemistry (e.g., legacy r9.4 450bps 6-mer model); wrong models degrade quantization and mapping
- FAST5 files may be VBZ-compressed, requiring the nanoporetech/vbz_compression HDF5 plugin before reading
- RawHash2's minimizer seeding mode is around 3X faster but slightly less accurate than non-minimizer mode, which affects downstream abundance precision
- Throughput advantages are genome-size dependent; accuracy is only on par (not better) for smaller genomes such as yeast or green algae

## Evidence

- [abstract] We evaluate RawHash on three applications: (i) read mapping, (ii) relative abundance estimation, and (iii) contamination analysis.: "We evaluate RawHash on three applications: (i) read mapping, (ii) relative abundance estimation, and (iii) contamination analysis."
- [methods] RawHash can be used to map reads from FAST5, POD5, SLOW5, or BLOW5 files: "RawHash can be used to map reads from **FAST5, POD5, SLOW5, or BLOW5** files to a reference genome in sequence format."
- [methods] We follow the similar PAF template used in UNCALLED: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED)"
- [methods] your reads in fast5 files are compressed with the VBZ compression from Nanopore: "your reads in fast5 files are compressed with the [VBZ compression](https://github.com/nanoporetech/vbz_compression) from Nanopore"
- [readme] Note that UNCALLED can only be applied to legacy r9.4.1 data: [evidence span withheld — anchor status: external_doc]
