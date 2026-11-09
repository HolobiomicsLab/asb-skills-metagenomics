---
name: benchmark-dataset-assembly-execution
description: Use when you need to produce de novo assemblies from benchmark Illumina
  short-read datasets to compare assembler versions or tools on N50, error counts,
  and correct assembly length.
when_to_use_negative:
- Input is single-cell or metagenomic data — the README directs such projects to MEGAHIT,
  which generates longer contigs with less memory, with SOAPdenovo-fusion for scaffolding
  its contigs
- The goal is comparing raw kmer/scaffolding modules rather than producing whole assemblies
  for benchmark scoring — use the stepwise module runs directly instead
- Read data are long reads (e.g. PacBio/ONT); SOAPdenovo2 is designed for Illumina
  GA short reads
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: SOAPdenovo2
  role: Primary assembler run on benchmark datasets (pregraph/sparse_pregraph, contig,
    map, scaff, GapCloser)
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo1.05
  role: Predecessor baseline run with comparable configuration on the same GAGE datasets
- name: GapCloser
  role: Gap closure module invoked via scaff -F to fill scaffold gaps using context
    and PE reads information
  repo: https://github.com/aquaskyline/SOAPdenovo2
- name: ALLPATHS-LG
  role: Competitor assembler whose results were compared against SOAPdenovo2 on Assemblathon1
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:207f4c0dfcb57ef64f640b6490d527401eda3f0769a283848c6dd43a78e77e2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/SOAPdenovo2
  tools:
  - ALLPATHS-LG
  - GapCloser
  - SOAPdenovo
  - SOAPdenovo1.05
  - SOAPdenovo2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# benchmark-dataset-assembly-execution

## Summary

Run a de novo assembler end-to-end on standard benchmark read sets (GAGE datasets such as S. aureus, R. sphaeroides, human chr14; Assemblathon1) using SOAPdenovo2's six-module pipeline so that assemblies can be scored against reference-based metrics and compared with predecessor or competitor assemblers.

## When to use

Use when you need to produce de novo assemblies from benchmark Illumina short-read datasets to compare assembler versions or tools on N50, error counts, and correct assembly length.

## When NOT to use

- Input is single-cell or metagenomic data — the README directs such projects to MEGAHIT, which generates longer contigs with less memory, with SOAPdenovo-fusion for scaffolding its contigs
- The goal is comparing raw kmer/scaffolding modules rather than producing whole assemblies for benchmark scoring — use the stepwise module runs directly instead
- Read data are long reads (e.g. PacBio/ONT); SOAPdenovo2 is designed for Illumina GA short reads

## Inputs

- Illumina GA short-read files in FASTA, FASTQ, or BAM format (paired-end and mate-pair libraries) from GAGE benchmark datasets: Staphylococcus aureus, Rhodobacter sphaeroides, human chr14 (also Assemblathon1)
- SOAPdenovo2 configuration file with [LIB] sections describing each library
- GAGE reference genomes for evaluation
- SOAPdenovo v1.05 executable as baseline

## Outputs

- *.contig — contig sequences assembled without mate-pair information
- *.scafSeq — scaffold sequences with gaps filled by GapCloser
- Assembly statistics: contig/scaffold N50, correct assembly length, structural/substitution/copy-number error counts vs. GAGE references
- Comparison table (e.g. gage_assembly_evaluation_table.csv) of SOAPdenovo2 vs SOAPdenovo1.05

## How to apply

Build SOAPdenovo2 from source with 'make' at the root of the unpacked folder (use 'make debug=1' only for debug builds). Write a configuration file with global max_rd_len plus [LIB] sections specifying avg_ins, reverse_seq, asm_flags, rd_len_cutoff, rank, pair_num_cutoff (minimum 3 for paired-end, 5 for mate-pair), and map_len (minimum 32 for PE, 35 for mate-pair). Run either the single 'all' command or the stepwise pipeline: pregraph (or sparse_pregraph with -z genomeSize) for sparse de Bruijn graph construction, contig for contig assembly, map for paired-end reads mapping, and scaff with -F to invoke GapCloser gap closure; a typical call uses -K 63 -R. Apply the same pipeline with comparable configuration to SOAPdenovo v1.05 as the predecessor baseline, then evaluate each assembly against GAGE references for correct assembly length increase, contig/scaffold N50, and structural, substitution, and copy-number error counts.

## Related tools

- **SOAPdenovo2** (Primary assembler run on benchmark datasets (pregraph/sparse_pregraph, contig, map, scaff, GapCloser)) — https://github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo1.05** (Predecessor baseline run with comparable configuration on the same GAGE datasets)
- **GapCloser** (Gap closure module invoked via scaff -F to fill scaffold gaps using context and PE reads information) — https://github.com/aquaskyline/SOAPdenovo2
- **ALLPATHS-LG** (Competitor assembler whose results were compared against SOAPdenovo2 on Assemblathon1)

## Examples

```
${bin} all -s config_file -K 63 -R -o graph_prefix 1>ass.log 2>ass.err
```

## Evaluation signals

- GAGE error reductions versus SOAPdenovo1 should reproduce reported values: structural error down 90.12%, substitution error down 92.13%, copy-number error down 69.47%
- Correct assembly length increase of ~3 to 80-fold over SOAPdenovo1 on GAGE datasets (Tables 2 and 3)
- Output files *.contig and *.scafSeq are produced for each dataset without pipeline-stage errors (ass.log/ass.err and per-step logs clean)
- On Assemblathon1, contig N50 should be ~1.53x and scaffold N50 ~1.84x longer than ALLPATHS-LG for cross-tool sanity
- Sparse_pregraph -z genomeSize should be set slightly larger than the real genome size for correct memory allocation

## Limitations

- Requires a 64-bit Linux system with minimum 5 GB physical memory; human-sized genomes need ~150 GB
- Sparse pregraph module usually needs 2-10 bp smaller kmer length than original pregraph for equivalent performance; K-mer must be odd, between 13 and 63/127 (README FAQ states 13-31)
- Reads longer than max_rd_len in the config are cut, and read length/depth constraints limit achievable kmer uniqueness for large genomes

## Evidence

- [results] GAGE datasets run via standard pipeline: "Improvements of SOAPdenovo2 have also been observed in assembling GAGE [8] dataset"
- [results] Six-module pipeline definition: "six modules that handle read error correction, de Bruijn graph (DBG) construction, contig assembly, paired-end (PE) reads mapping, scaffold construction, and gap closure"
- [abstract] Benchmark superiority over predecessor: "Benchmark using the Assemblathon1 and GAGE datasets showed that SOAPdenovo2 greatly surpasses its predecessor SOAPdenovo"
- [readme] Build command from README: [evidence span withheld — anchor status: external_doc]
- [readme] Typical run command from README: [evidence span withheld — anchor status: external_doc]
- [readme] Supported read formats: [evidence span withheld — anchor status: external_doc]
