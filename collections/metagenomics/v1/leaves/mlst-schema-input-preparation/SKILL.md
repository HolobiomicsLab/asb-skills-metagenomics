---
name: mlst-schema-input-preparation
description: Use when you already possess a chewBBACA-created or adapted cgMLST/wgMLST
  schema (schema_seed folder) plus draft genome FASTA files, and must stage and validate
  these inputs before running chewBBACA.py AlleleCall.
when_to_use_negative:
- Input is raw sequencing reads rather than draft/complete genome FASTA assemblies
  — chewBBACA's allele-calling engine takes FASTA assemblies, while read-based callers
  like MentaLiST handle reads
- No schema exists yet and loci must first be defined de novo — this skill stages
  an existing schema_seed, it does not perform CreateSchema
- The schema was produced by another platform and has not been adapted — first run
  PrepExternalSchema (backing up the source FASTA, since it is modified)
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: chewBBACA
  role: Provides CreateSchema (schema_seed generation), PrepExternalSchema (adapting
    external EnteroBase/PubMLST/Ridom schemas) and the AlleleCall engine that consumes
    the staged schema and assemblies
  repo: github.com/B-UMMI/chewBBACA
- name: BLAST
  role: Underlies the BSR-based allele calling performed by chewBBACA's AlleleCall
    over the prepared loci and genomes
- name: Python 3.4
  role: Runtime requirement for executing chewBBACA on a laptop or HPC cluster
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:1e89ce7aceeb58c2cde695bcd8dcca62bc133a7b1e376e315c80edadce3df29b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - Python 3.4
  - chewBBACA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# mlst-schema-input-preparation

## Summary

Prepares the schema and assembly inputs required for chewBBACA's BLAST-based allele calling: a schema seed directory of representative loci (optionally restricted by a gene list), a species Prodigal training file, and FASTA draft/complete genome assemblies that are fed to the AlleleCall engine.

## When to use

Use when you already possess a chewBBACA-created or adapted cgMLST/wgMLST schema (schema_seed folder) plus draft genome FASTA files, and must stage and validate these inputs before running chewBBACA.py AlleleCall.

## When NOT to use

- Input is raw sequencing reads rather than draft/complete genome FASTA assemblies — chewBBACA's allele-calling engine takes FASTA assemblies, while read-based callers like MentaLiST handle reads
- No schema exists yet and loci must first be defined de novo — this skill stages an existing schema_seed, it does not perform CreateSchema
- The schema was produced by another platform and has not been adapted — first run PrepExternalSchema (backing up the source FASTA, since it is modified)

## Inputs

- cgMLST/wgMLST schema seed directory of loci FASTA files (one representative allele per locus)
- FASTA files of draft or complete genome assemblies (e.g. 100 S. agalactiae assemblies from FigShare)
- Prodigal training file for the species (e.g. Streptococcus_agalactiae.trn)
- Optional gene list file restricting calling to core loci (e.g. cgMLSTschema.txt at 95% threshold)

## Outputs

- Allelic profile matrix (results_alleles.tsv) mapping strains to allele identifiers
- Updated schema FASTA files containing novel alleles appended during calling
- RepeatedLoci.txt listing paralogous loci
- Ready-to-run AlleleCall invocation over the staged inputs

## How to apply

First confirm the schema exists as a directory of loci FASTAs with a single representative allele per locus (the schema_seed produced by CreateSchema or by PrepExternalSchema for external formats such as EnteroBase, PubMLST or Ridom). Gather the input assemblies as FASTA files of draft or complete genomes (e.g. the S. agalactiae assemblies from FigShare) and, for Streptococcus agalactiae, attach the provided Prodigal training file (Streptococcus_agalactiae.trn). Optionally build a gene list (e.g. the cgMLST_95/cgMLSTschema.txt file of loci present in >=95% of genomes after removing the 20 paralogous loci reported in RepeatedLoci.txt) to pass to --gl so allele calling is restricted to core loci. Then invoke AlleleCall with -i (assemblies), -g (schema seed) and -o (output), keeping the default BSR threshold of 0.6; the engine performs BLAST-based allele calls and assigns identifiers to novel alleles, adding them to the schema. Before running, back up external schemas because PrepExternalSchema modifies the original source FASTA files.

## Related tools

- **chewBBACA** (Provides CreateSchema (schema_seed generation), PrepExternalSchema (adapting external EnteroBase/PubMLST/Ridom schemas) and the AlleleCall engine that consumes the staged schema and assemblies) — github.com/B-UMMI/chewBBACA
- **BLAST** (Underlies the BSR-based allele calling performed by chewBBACA's AlleleCall over the prepared loci and genomes)
- **Python 3.4** (Runtime requirement for executing chewBBACA on a laptop or HPC cluster)

## Examples

```
chewBBACA.py AlleleCall -i complete_genomes/ -g tutorial_schema/schema_seed -o results32_wgMLST --cpu 6
```

## Evaluation signals

- schema_seed directory contains one representative allele per locus; a CreateSchema run on 32 S. agalactiae complete genomes should yield ~3128 loci
- AlleleCall invoked with the default BSR threshold of 0.6 completes in seconds to minutes per strain on a laptop (tutorial average ~32 s per genome)
- Output allelic profile matrix is produced alongside RepeatedLoci.txt; novel alleles are appended to the schema (e.g. 3,128 to 17,848 alleles in the tutorial)
- If a --gl gene list was used, profile columns match exactly the core loci selected at the chosen presence threshold (95%)
- External schema inputs are backed up before PrepExternalSchema, since the function modifies the original source FASTA files

## Limitations

- PrepExternalSchema modifies the original source FASTA, so users must back up data before using this function
- Paralogous loci listed in RepeatedLoci.txt should be removed from profiles before downstream cgMLST definition, due to potential uncertainty in allele assignment
- Allele calling runtime scales near-linearly only up to ~32 CPUs on the HPC benchmark; beyond that disk/I/O becomes the bottleneck, so very large CPU allocations do not improve throughput
- For external schemas from platforms like EnteroBase, no stand-alone allele-calling algorithm exists and reads would need submission to public repositories unless adapted via PrepExternalSchema

## Evidence

- [abstract] abstract: "chewBBACA performs the schema creation and allele calls on complete or draft genomes resulting from de novo assemblers."
- [readme] readme: [evidence span withheld — anchor status: not_found]
- [readme] readme_2: [evidence span withheld — anchor status: external_doc]
- [discussion] discussion: "The allele-calling engine of chewBBACA uses FASTA files with draft assemblies or complete genomes as input"
- [intro] intro: "typically using an approach based on blast [4]"
