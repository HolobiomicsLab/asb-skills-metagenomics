---
name: wgmlst-schema-construction
description: Use when the input is a collection of draft or complete genome FASTA
  assemblies and the goal is to define a new wgMLST or cgMLST schema seed from the
  distinct loci present in those genomes, prior to allele calling.
when_to_use_negative:
- Input is raw sequencing reads rather than de novo assemblies — chewBBACA's schema
  creation and allele calling operate on complete or draft genome FASTA files
- A pre-existing or external schema already defines the loci (use AlleleCall against
  the existing schema, or PrepExternalSchema to adapt external schemas from platforms
  like EnteroBase, PubMLST, or Ridom cgMLST)
- The goal is allele calling only against an already-built schema seed rather than
  de novo locus definition
edam_operation: http://edamontology.org/operation_0291
tools:
- name: chewBBACA
  role: Suite providing the CreateSchema module that performs CDS prediction, filtering,
    and BSR-based clustering to build the wg/cgMLST schema seed
  repo: https://github.com/B-UMMI/chewBBACA
- name: Prodigal
  role: CDS prediction engine invoked by CreateSchema on each input draft or complete
    genome assembly, guided by a species training file
  repo: https://github.com/B-UMMI/chewBBACA
- name: BLAST
  role: All-against-all BLASTP search on translated CDSs used to compute pairwise
    similarity and BLAST Score Ratios for locus clustering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:bdb94cb53332d08ed8639c3da4284405c2d704cd48c84ce7dc1cf245401c441d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - BLAST
  - Prodigal
  - chewBBACA
  edam_operation: http://edamontology.org/operation_0291
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# wgmlst-schema-construction

## Summary

Constructs a novel wg/cgMLST schema seed from a set of draft or complete bacterial genome assemblies using chewBBACA's CreateSchema, which predicts CDSs with Prodigal and clusters translated CDSs into loci via all-against-all BLASTp using a BLAST Score Ratio (BSR) threshold of 0.6.

## When to use

Use when the input is a collection of draft or complete genome FASTA assemblies and the goal is to define a new wgMLST or cgMLST schema seed from the distinct loci present in those genomes, prior to allele calling.

## When NOT to use

- Input is raw sequencing reads rather than de novo assemblies — chewBBACA's schema creation and allele calling operate on complete or draft genome FASTA files
- A pre-existing or external schema already defines the loci (use AlleleCall against the existing schema, or PrepExternalSchema to adapt external schemas from platforms like EnteroBase, PubMLST, or Ridom cgMLST)
- The goal is allele calling only against an already-built schema seed rather than de novo locus definition

## Inputs

- Directory of draft or complete genome assemblies in FASTA format (e.g. 32 complete S. agalactiae genomes)
- Species-specific Prodigal training file (.trn), e.g. Streptococcus_agalactiae.trn
- Optional parameters: --cpu core count

## Outputs

- schema_seed directory with one FASTA file per locus containing the representative allele (schema seed with one allele per locus)
- Downstream: allelic profile matrix (results_alleles.tsv) and per-locus FASTA files of allelic diversity after AlleleCall
- RepeatedLoci.txt listing candidate paralogous loci

## How to apply

Run chewBBACA CreateSchema on the assembly directory, passing a species-specific Prodigal training file (--ptf, e.g. Streptococcus_agalactiae.trn) and a --cpu count matched to the machine. CreateSchema runs Prodigal CDS prediction on each input draft or complete genome, removes duplicated shorter copies of identical CDSs and CDSs below a minimum length threshold, then performs an all-against-all BLASTP search on the remaining translated CDSs. CDSs whose BLAST Score Ratio is >= 0.6 are merged as alleles of the same locus, retaining the larger allele as the representative sequence. The output is the schema_seed directory containing one FASTA file per locus, each with a single representative allele; the tutorial reports 3128 loci from 32 Streptococcus agalactiae complete genomes. After schema construction, run AlleleCall against the schema seed to capture allelic diversity, remove paralogous loci listed in RepeatedLoci.txt with RemoveGenes, and derive the core genome with ExtractCgMLST at a chosen presence threshold (e.g. 0.95).

## Related tools

- **chewBBACA** (Suite providing the CreateSchema module that performs CDS prediction, filtering, and BSR-based clustering to build the wg/cgMLST schema seed) — https://github.com/B-UMMI/chewBBACA
- **Prodigal** (CDS prediction engine invoked by CreateSchema on each input draft or complete genome assembly, guided by a species training file) — https://github.com/B-UMMI/chewBBACA
- **BLAST** (All-against-all BLASTP search on translated CDSs used to compute pairwise similarity and BLAST Score Ratios for locus clustering)

## Examples

```
chewBBACA.py CreateSchema -i complete_genomes/ -o tutorial_schema --ptf Streptococcus_agalactiae.trn --cpu 6
```

## Evaluation signals

- tutorial_schema/schema_seed exists and contains one valid FASTA file per schema locus (3128 loci for the tutorial's 32 S. agalactiae complete genomes)
- Each schema locus FASTA holds a single representative allele — at schema-creation time the schema is 'a set of loci each with a single representative allele'
- All retained alleles satisfy the clustering criterion (BSR >= 0.6; CreateSchema merges CDSs with BSR >= 0.6 into the same locus)
- Downstream AlleleCall with the schema seed and default BSR threshold of 0.6 succeeds and reports novel alleles (tutorial: 14,720 novel alleles on the 32 complete genomes)
- Paralogous loci flagged in RepeatedLoci.txt (20 loci in the tutorial) are identified and removed before defining the core genome

## Limitations

- Paralogous loci reported in RepeatedLoci.txt should be removed from the schema due to potential uncertainty in allele assignment (20 loci in the tutorial)
- Core-genome loci may be missed because of sequencing coverage and assembly problems; a 95% presence threshold was chosen to account for this
- PrepExternalSchema (schema adaptation) modifies the original source FASTA, requiring users to back up data beforehand
- Creating a schema from a single FASTA file skips the gene prediction step, changing the analysis path

## Evidence

- [intro] chewBBACA performs the schema creation and allele calls on complete or draft genomes resulting from de novo assemblers: "chewBBACA performs the schema creation and allele calls on complete or draft genomes resulting from de novo assemblers"
- [intro] We developed chewBBACA to be a complete stand-alone pipeline for GbG analyses, including constructing and validating novel cg/wgMLST schemas: "We developed chewBBACA to be a complete stand-alone pipeline for GbG analyses, including constructing and validating novel cg/wgMLST schemas"
- [readme] chewBBACA.py CreateSchema -i complete_genomes/ -o tutorial_schema --ptf Streptococcus_agalactiae.trn --cpu 6: [evidence span withheld — anchor status: external_doc]
- [readme] In our system, the process took 56 seconds to complete resulting on a wgMLST schema with 3128 loci. At this point the schema is defined as a set of loci each with a single representative allele.: [evidence span withheld — anchor status: external_doc]
- [readme] The allele call used the default BSR threshold of 0.6: [evidence span withheld — anchor status: external_doc]
- [readme] These loci should be removed from the schema due to the potential uncertainty in allele assignment: [evidence span withheld — anchor status: external_doc]
