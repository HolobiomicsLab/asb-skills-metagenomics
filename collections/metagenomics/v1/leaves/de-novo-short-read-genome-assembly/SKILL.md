---
name: de-novo-short-read-genome-assembly
description: Use when you have trimmed paired-end Illumina reads from an isolate genome
  (e.g., E. coli K-12 MG1655, ~4.6-4.8 Mb) and need de novo contig assemblies from
  several widely-cited short-read assemblers for downstream quality comparison.
when_to_use_negative:
- Long-read or hybrid sequencing data is the input — these assemblers were shortlisted
  and benchmarked as short-read assemblers only
- A high-quality reference genome is available and a reference-guided assembly or
  variant-aware evaluation (e.g., against a close substrain such as W3110) is the
  actual goal
- The goal is assembly quality assessment rather than assembly generation — use QUAST/WebQUAST
  for evaluation instead
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ABySS
  role: Short-read assembler run on trimmed paired-end reads using the GAGE-B parameter
    recipe because its default assembly was of very poor quality; produced the largest
    (4.8 Mb) but most fragmented assembly (176 contigs)
- name: MEGAHIT
  role: Short-read assembler run with default parameters to produce a contig FASTA
    (~90-95 contigs, largest contig 236 kb)
- name: SPAdes
  role: Short-read assembler run with default/recommended parameters; produced the
    best N50, N90 and auN values and the largest contig overall (285 kb)
- name: Velvet
  role: Short-read assembler run as velveth followed by velvetg with documentation-recommended
    parameters (~90-95 contigs, largest contig 265 kb)
- name: QUAST
  role: Downstream evaluator used to verify assembly quality (N50, N90, auN, total
    length, contig counts) of the generated FASTA files
  repo: https://github.com/ablab/quast
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:7e118f04e91a3703f175b317a866928a1430edc67846a3ec6153efcff6134f22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - ABySS
  - MEGAHIT
  - QUAST
  - SPAdes
  - Velvet
  - WebQUAST
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# de-novo-short-read-genome-assembly

## Summary

Assemble trimmed paired-end short reads into a contig FASTA using multiple short-read assemblers (ABySS, MEGAHIT, SPAdes, Velvet installed via Bioconda), as demonstrated on the E. coli K-12 MG1655 dataset (SRA ERR008613) used to build WebQUAST's sample evaluation dataset.

## When to use

Use when you have trimmed paired-end Illumina reads from an isolate genome (e.g., E. coli K-12 MG1655, ~4.6-4.8 Mb) and need de novo contig assemblies from several widely-cited short-read assemblers for downstream quality comparison.

## When NOT to use

- Long-read or hybrid sequencing data is the input — these assemblers were shortlisted and benchmarked as short-read assemblers only
- A high-quality reference genome is available and a reference-guided assembly or variant-aware evaluation (e.g., against a close substrain such as W3110) is the actual goal
- The goal is assembly quality assessment rather than assembly generation — use QUAST/WebQUAST for evaluation instead

## Inputs

- Trimmed paired-end short-read FASTQ files (e.g., E. coli K-12 MG1655 reads, NCBI SRA accession ERR008613)
- Assembler parameter recipes (default parameters for MEGAHIT/SPAdes/Velvet; GAGE-B recipe for ABySS)

## Outputs

- One contig FASTA file per assembler (four total: ABySS, MEGAHIT, SPAdes, Velvet)
- Assembly size and contig-count summary in the expected ~4.6-4.8 Mb total length and 90-176 contig ranges

## How to apply

Install ABySS, MEGAHIT, SPAdes and Velvet from Bioconda into a conda environment; these four tools were shortlisted as the most-cited short-read assemblers in the WebQUAST paper. Run MEGAHIT, SPAdes and Velvet (velveth followed by velvetg) on the trimmed paired-end reads with default or documentation-recommended parameters to produce contig FASTA files. Run ABySS with the GAGE-B parameter recipe (which evaluates multiple k-mer sizes and selects the best assembly) rather than its defaults, because the paper found ABySS's default assembly to be of very poor quality. Confirm each of the four assembly FASTA files exists and that total lengths fall in the ~4.6-4.8 Mb range before handing them to an evaluator such as QUAST/WebQUAST.

## Related tools

- **ABySS** (Short-read assembler run on trimmed paired-end reads using the GAGE-B parameter recipe because its default assembly was of very poor quality; produced the largest (4.8 Mb) but most fragmented assembly (176 contigs))
- **MEGAHIT** (Short-read assembler run with default parameters to produce a contig FASTA (~90-95 contigs, largest contig 236 kb))
- **SPAdes** (Short-read assembler run with default/recommended parameters; produced the best N50, N90 and auN values and the largest contig overall (285 kb))
- **Velvet** (Short-read assembler run as velveth followed by velvetg with documentation-recommended parameters (~90-95 contigs, largest contig 265 kb))
- **QUAST** (Downstream evaluator used to verify assembly quality (N50, N90, auN, total length, contig counts) of the generated FASTA files) — https://github.com/ablab/quast

## Examples

```
conda install -c bioconda abyss megahit spades velvet
```

## Evaluation signals

- Each of the four assembler runs yields a non-empty contig FASTA file
- Total assembly lengths fall in the ~4.6-4.8 Mb range reported for the E. coli K-12 MG1655 assemblies
- Contig counts fall within the 90-176 range: Velvet, SPAdes and MEGAHIT produce ~90-95 contigs while ABySS is the most fragmented at 176 contigs
- Largest-contig sizes match the reported ordering: SPAdes 285 kb, Velvet 265 kb, ABySS 248 kb, MEGAHIT 236 kb
- QUAST summary metrics (N50, N90, auN) confirm SPAdes has the best contiguity among the four assemblies

## Limitations

- ABySS's default parameters give a very poor quality assembly for this dataset; the GAGE-B parameter recipe (multi-k evaluation) is required
- Results are demonstrated only on a small model genome (E. coli K-12 MG1655, ~4.6-4.8 Mb); behavior on larger or more repetitive genomes is not established by this workflow
- Assemblers rely on different heuristics, so even different settings of the same tool may result in substantially diverging assemblies, requiring comparative evaluation before any assembly is selected

## Evidence

- [methods] MEGAHIT, SPAdes, and Velvet ran with default or documentation-recommended parameters, while ABySS used the GAGE-B recipe because its default assembly was of very poor quality.: [evidence span withheld — anchor status: not_found]
- [results] ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT): "ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT)"
- [supplementary] The sequencing data for E. coli K-12 MG1655 dataset is available from the National Center for Biotechnology Information (NCBI) Sequence Read Archive under accession number ERR008613: "The sequencing data for E. coli K-12 MG1655 dataset is available from the National Center for Biotechnology Information (NCBI) Sequence Read Archive under accession number ERR008613"
- [readme] QUAST stands for QUality ASsessment Tool. It evaluates genome/metagenome assemblies by computing various metrics.: "QUAST stands for QUality ASsessment Tool. It evaluates genome/metagenome assemblies by computing various metrics."
