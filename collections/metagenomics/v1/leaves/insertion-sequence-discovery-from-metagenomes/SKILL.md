---
name: insertion-sequence-discovery-from-metagenomes
description: Use when the input is paired-end short-read metagenomic FASTQ.GZ data
  from mixed microbial communities (e.g. HMP oral/stool samples) and the goal is to
  detect ISs and build a non-redundant IS catalogue with novel transposase annotation.
when_to_use_negative:
- The microbial genomes/ISs of interest lack inverted terminal repeats — Palidis is
  'currently only equipped with discovering ISs with ITRs' and the ISC only contains
  common DDE types of ISs with ITRs
- ISs have ITRs shorter than 25 nt — the default minimum ITR length misses ITRs as
  short as ten nucleotides
- Input is isolate genomes only where a prebuilt catalogue suffices for annotation
  — use ISC v1.0.0 directly (it is recommended as reference for isolates from healthy
  human oral cavity and stool, the sources it was built from)
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Palidis (palidis4)
  role: Nextflow pipeline that discovers ISs from metagenomic reads by identifying
    ITRs via maximal exact matching, mapping reads with Bowtie2, filtering candidate
    ITRs by distance, and clustering with CD-HIT-EST
  repo: https://github.com/blue-moon22/palidis4
- name: pal-MEM
  role: Efficient Maximal Exact Match computation used inside Palidis to find inverted
    terminal repeat regions in large metagenomic read files (modified from E-MEM)
  repo: https://github.com/blue-moon22/pal-MEM
- name: CD-HIT-EST
  role: Clusters candidate ITRs within Palidis (clusterReads step) and clusters pooled
    per-sample ISs at -c 0.95 to produce the non-redundant catalogue
- name: ISC (Insertion Sequence Catalogue)
  role: Released catalogue output (v1.0.0, 879 ISs) available as a reference for annotating
    isolate genomes
  repo: https://github.com/blue-moon22/ISC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:5e827e6d38a2f09fc1d754ea7c444a634fa83503943a2047b5704693c6a0c10e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis4
  tools:
  - CD-HIT-EST
  - ISC (Insertion Sequence Catalogue)
  - Palidis
  - Palidis (palidis4)
  - pal-MEM
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# insertion-sequence-discovery-from-metagenomes

## Summary

Discover novel insertion sequences (ISs) in short-read metagenomic data using Palidis, which detects palindromic inverted terminal repeats (ITRs) via maximal exact matching, then pools and clusters detections into a non-redundant catalogue (e.g. ISC v1.0.0 with 879 unique ISs).

## When to use

Use when the input is paired-end short-read metagenomic FASTQ.GZ data from mixed microbial communities (e.g. HMP oral/stool samples) and the goal is to detect ISs and build a non-redundant IS catalogue with novel transposase annotation.

## When NOT to use

- The microbial genomes/ISs of interest lack inverted terminal repeats — Palidis is 'currently only equipped with discovering ISs with ITRs' and the ISC only contains common DDE types of ISs with ITRs
- ISs have ITRs shorter than 25 nt — the default minimum ITR length misses ITRs as short as ten nucleotides
- Input is isolate genomes only where a prebuilt catalogue suffices for annotation — use ISC v1.0.0 directly (it is recommended as reference for isolates from healthy human oral cavity and stool, the sources it was built from)

## Inputs

- Paired-end short-read metagenomic FASTQ.GZ files (e.g. Human Microbiome Project raw reads with download links in Data S1)
- Per-sample assembled contigs in FASTA format (contigs_path in the manifest)
- Tab-delimited manifest with headers lane_id, read_directory, sample_id, contigs_path

## Outputs

- FASTA file of insertion sequences per sample
- Table of per-IS information (IS_name, sample_id, contig, is_start, is_end, description with InterPro/PANTHER transposase accessions)
- Non-redundant IS catalogue (e.g. Insertion Sequence Catalogue v1.0.0 with 879 unique ISs)

## How to apply

Run Palidis (Nextflow pipeline palidis4, default parameters: minimum ITR length 25 nt, k-mer length 15 for maximal exact matching via pal-MEM, IS length 500-3000 bp) on each metagenome with a tab-delimited manifest giving lane_id, read_directory, sample_id and contigs_path. Palidis pre-processes FASTQ.GZ reads with convertToFasta, finds repeat sequences with pal-MEM, maps reads against assemblies using Bowtie2, applies distance filters to get candidate ITRs, clusters candidate ITRs with CD-HIT-EST, and annotates transposases via Prodigal and InterProScan. Pool per-sample IS FASTA outputs and cluster them with CD-HIT-EST at a sequence identity threshold of -c 0.95 to remove redundancy and emit the non-redundant catalogue. For the HMP application this yielded 2517 ISs on 1837 contigs across 218 of 264 samples, clustering to 879 unique ISs with 87 unique transposases.

## Related tools

- **Palidis (palidis4)** (Nextflow pipeline that discovers ISs from metagenomic reads by identifying ITRs via maximal exact matching, mapping reads with Bowtie2, filtering candidate ITRs by distance, and clustering with CD-HIT-EST) — https://github.com/blue-moon22/palidis4
- **pal-MEM** (Efficient Maximal Exact Match computation used inside Palidis to find inverted terminal repeat regions in large metagenomic read files (modified from E-MEM)) — https://github.com/blue-moon22/pal-MEM
- **CD-HIT-EST** (Clusters candidate ITRs within Palidis (clusterReads step) and clusters pooled per-sample ISs at -c 0.95 to produce the non-redundant catalogue)
- **ISC (Insertion Sequence Catalogue)** (Released catalogue output (v1.0.0, 879 ISs) available as a reference for annotating isolate genomes) — https://github.com/blue-moon22/ISC

## Examples

```
nextflow palidis.nf --manifest manifest.tsv --batch_name my_batch -c configs/conf/my_hpc.config --min_itr_length 25 --min_is_len 500 --max_is_len 3000
```

## Evaluation signals

- Applying Palidis v3.1.0 defaults to the 264 HMP metagenomes yields 2517 ISs on 1837 contigs across 218 of 264 samples
- CD-HIT-EST clustering (-c 0.95) of pooled ISs produces exactly 879 unique insertion sequences with 87 unique transposases, matching released ISC v1.0.0 (github.com/blue-moon22/ISC)
- 519 of the 879 ISs should be novel, i.e. not previously characterized (e.g. absent from ISfinder)
- All reported ISs must be flanked by palindromic ITRs of >=25 nt and fall within the 500-3000 bp IS length range
- palidis4 regression test passes: './tests/regression_tests.sh' prints 'Test passed. All outputs expected.'

## Limitations

- ISC only contains common DDE types of ISs with ITRs; Palidis cannot discover ISs lacking ITRs
- The catalogue contains ISs with ITRs of 25 or greater nucleotides, although ITRs can be as short as ten nucleotides; lowering the minimum ITR length and k-mer length is more computationally intensive, especially with complex mixed microbial genomes
- ISs flanked by ITRs beyond the 500 to 3000 bp proximity range (e.g. regions lengthened by nested insertion events) are missed; increasing maximum IS length may be done in future ISC iterations
- ISC v1.0.0 was built only from healthy human oral cavity and stool metagenomes, so it is recommended as an annotation reference only for isolates sourced from those environments

## Evidence

- [abstract] Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel and not previously characterized.: "Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel and not previously characterized."
- [intro] Palidis ITR detection method: "we present a tool called Palidis (palindromic detection of insertion sequences) that finds ISs using an efficient maximal exact matching algorithm"
- [readme] Palidis pipeline steps in README: [evidence span withheld — anchor status: not_found]
- [readme] Palidis default parameters from README: "--min_itr_length    Minimum length of ITR. (Default: 25) --max_itr_length    Maximum length of ITR. (Default: 50) --kmer_length       k-mer length for maximal exact matching. (Default: 15)"
- [discussion] ITR length limitation from discussion: "the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis, although ITRs can be as short as ten nucleotides in length"
- [full_text] Workflow validation from source card: [evidence span withheld — anchor status: not_found]
