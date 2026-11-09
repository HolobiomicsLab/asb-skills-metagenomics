---
name: assembler-parameter-recipe-execution
description: Use when reproducing the four E. coli K-12 MG1655 sample assemblies from
  trimmed paired-end FASTQ reads (SRA ERR008613) as input for WebQUAST/QUAST assembly
  evaluation, or when comparing short-read assemblers under their citation-standard
  parameter settings.
when_to_use_negative:
- The input is already assembled contigs — this skill generates assemblies from trimmed
  reads; feed existing FASTA files directly to WebQUAST/QUAST instead
- The target organism or read data is not short-read (e.g. long-read or hybrid data)
  — the recipes here are for the four shortlisted short-read assemblers on Illumina-style
  paired-end reads
- A single assembler with custom k-mer optimization is required — this skill executes
  documented default/recommended settings or the GAGE-B recipe, not bespoke parameter
  tuning
edam_operation: http://edamontology.org/operation_0310
edam_topics: []
tools:
- name: ABySS
  role: Runs on trimmed paired-end reads using the GAGE-B parameter recipe (multiple
    k-mer sizes, best assembly selected) because its default assembly is of very poor
    quality
- name: MEGAHIT
  role: Runs on trimmed reads with default parameters to produce a contig FASTA
- name: SPAdes
  role: Runs on trimmed reads with default/recommended parameters to produce a contig
    FASTA
- name: Velvet
  role: Runs as velveth followed by velvetg on trimmed reads with recommended parameters
    to produce a contig FASTA
- name: WebQUAST
  role: Downstream consumer of the four generated assemblies for quality assessment
    and comparison
  repo: https://github.com/ablab/quast-website
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:1e5f78b9bd78c9bb3691f4a96488d2ea21684a641c9294261c64f140778a398b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast-website
  tools:
  - ABySS
  - MEGAHIT
  - SPAdes
  - Velvet
  - WebQUAST
  edam_operation: http://edamontology.org/operation_0310
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# assembler-parameter-recipe-execution

## Summary

Reconstructs the four-assembler E. coli K-12 MG1655 demonstration assemblies (ABySS, MEGAHIT, SPAdes, Velvet) by installing each assembler via Bioconda and running them on trimmed paired-end reads with default or recipe-specific parameters, producing contig FASTA files whose properties match the assemblies evaluated with WebQUAST.

## When to use

Use when reproducing the four E. coli K-12 MG1655 sample assemblies from trimmed paired-end FASTQ reads (SRA ERR008613) as input for WebQUAST/QUAST assembly evaluation, or when comparing short-read assemblers under their citation-standard parameter settings.

## When NOT to use

- The input is already assembled contigs — this skill generates assemblies from trimmed reads; feed existing FASTA files directly to WebQUAST/QUAST instead
- The target organism or read data is not short-read (e.g. long-read or hybrid data) — the recipes here are for the four shortlisted short-read assemblers on Illumina-style paired-end reads
- A single assembler with custom k-mer optimization is required — this skill executes documented default/recommended settings or the GAGE-B recipe, not bespoke parameter tuning

## Inputs

- Trimmed paired-end FASTQ reads from the E. coli K-12 MG1655 dataset (NCBI SRA accession ERR008613)
- Bioconda conda environment for tool installation

## Outputs

- Four contig FASTA assemblies (one each from ABySS, MEGAHIT, SPAdes, Velvet) suitable for evaluation with WebQUAST

## How to apply

Install the four shortlisted assemblers (ABySS, MEGAHIT, SPAdes, Velvet) from Bioconda into a conda environment. Run MEGAHIT on the trimmed paired-end reads with default parameters and SPAdes with default/documentation-recommended parameters, each producing a contig FASTA. Run Velvet as the two-step velveth followed by velvetg with recommended parameters. For ABySS, do NOT use defaults — its default assembly is of very poor quality — instead apply the GAGE-B parameter recipe, which evaluates multiple k-mer sizes and selects the best assembly. The asymmetry in ABySS's settings is the key decision point: it is the reason ABySS is the largest but most fragmented output, so validate accordingly rather than assuming all four used identical recipes.

## Related tools

- **ABySS** (Runs on trimmed paired-end reads using the GAGE-B parameter recipe (multiple k-mer sizes, best assembly selected) because its default assembly is of very poor quality)
- **MEGAHIT** (Runs on trimmed reads with default parameters to produce a contig FASTA)
- **SPAdes** (Runs on trimmed reads with default/recommended parameters to produce a contig FASTA)
- **Velvet** (Runs as velveth followed by velvetg on trimmed reads with recommended parameters to produce a contig FASTA)
- **WebQUAST** (Downstream consumer of the four generated assemblies for quality assessment and comparison) — https://github.com/ablab/quast-website

## Examples

```
conda install -c bioconda abyss megahit spades velvet
```

## Evaluation signals

- Each of the four assembly FASTA files exists and total assembly length falls in the ~4.6–4.8 Mb range
- Contig counts fall in the 90–176 range: Velvet, SPAdes and MEGAHIT produce ~90–95 contigs; ABySS is the most fragmented at 176 contigs
- ABySS yields the largest assembly (~4.8 Mb versus ~4.6 Mb for the others)
- SPAdes has the best N50, N90 and auN values and the largest single contig (285 kb versus 265, 248 and 236 kb for Velvet, ABySS and MEGAHIT)

## Limitations

- ABySS's default parameterization produces a very poor assembly, so the GAGE-B recipe (multi-k evaluation with selection of the best assembly) must be used; a naive default run will not reproduce the reported result
- Assembler outputs greatly depend on heuristics and settings — even different settings of the same tool can yield substantially diverging assemblies, so results are parameterization-dependent
- The reported contig counts and lengths (90–176 contigs, 4.6–4.8 Mb) are specific to the E. coli K-12 MG1655 ERR008613 dataset and will not transfer to other genomes

## Evidence

- [other] The four E. coli assemblies were produced with five most-cited short-read assemblers shortlisted to four tools, all installed via Bioconda.: [evidence span withheld — anchor status: not_found]
- [other] ABySS GAGE-B rationale: [evidence span withheld — anchor status: not_found]
- [results] Expected assembly properties: "ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT)"
- [results] SPAdes contiguity expectation: "SPAdes assembled larger contigs on average (the best N50, N90 and auN, the area under the Nx curve, values with Velvet and MEGAHIT being close runner-ups)"
- [supplementary] Dataset provenance: "The sequencing data for E. coli K-12 MG1655 dataset is available from the National Center for Biotechnology Information (NCBI) Sequence Read Archive under accession number ERR008613"
- [readme] Downstream evaluation context: "QUAST stands for QUality ASsessment Tool. It evaluates genome/metagenome assemblies by computing various metrics."
